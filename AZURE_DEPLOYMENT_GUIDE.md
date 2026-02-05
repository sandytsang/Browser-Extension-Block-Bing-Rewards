# Azure Deployment Guide for Self-Hosted Chrome Extension

This guide will help you deploy your Chrome extension on Azure for self-hosting on Linux systems.

## Prerequisites

- Azure account with active subscription
- Azure CLI installed (optional, can use Azure Portal)
- The `main.crx` file
- Extension ID (found in Chrome at `chrome://extensions/` after loading unpacked extension)

## Step 1: Get Your Extension ID

1. Open Chrome and go to `chrome://extensions/`
2. Enable "Developer mode"
3. Load the unpacked extension (use the extension folder)
4. Copy the Extension ID (it's a 32-character string like `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa`)
5. **IMPORTANT**: Save this ID, you'll need it for the manifest.json and updates.xml

## Step 2: Update Configuration Files

### Update manifest.json

Add the `update_url` field to your `manifest.json`:

```json
{
  "manifest_version": 3,
  "name": "Bing Rewards Blocker",
  "version": "1.0.0",
  "update_url": "https://YOUR_AZURE_DOMAIN.azurewebsites.net/updates.xml",
  ...rest of the manifest
}
```

**Note**: Replace `YOUR_AZURE_DOMAIN` with your actual Azure Web App name.

### Update updates.xml

Replace placeholders in `updates.xml`:
- Replace `YOUR_EXTENSION_ID_HERE` with your actual extension ID
- Replace `YOUR_AZURE_DOMAIN` with your Azure Web App name

## Step 3: Choose Your Azure Hosting Option

### Option A: Azure Static Web Apps (Recommended for simple hosting)

**Pros**: Free tier available, automatic HTTPS, CDN
**Cons**: Limited to static files

1. Go to Azure Portal: https://portal.azure.com
2. Click "Create a resource"
3. Search for "Static Web App"
4. Click "Create"
5. Fill in details:
   - **Subscription**: Select your subscription
   - **Resource Group**: Create new or select existing
   - **Name**: `bing-rewards-blocker-extension` (or your preferred name)
   - **Plan type**: Free
   - **Region**: Choose closest to your users
   - **Deployment**: Choose "Other" (manual deployment)
6. Click "Review + create" then "Create"

### Option B: Azure App Service (More flexible)

**Pros**: More configuration options, supports various platforms
**Cons**: No free tier for production

1. Go to Azure Portal: https://portal.azure.com
2. Click "Create a resource"
3. Search for "Web App"
4. Click "Create"
5. Fill in details:
   - **Subscription**: Select your subscription
   - **Resource Group**: Create new or select existing
   - **Name**: `bing-rewards-blocker` (this will be your URL)
   - **Publish**: Code
   - **Runtime stack**: Node.js (or Python/PHP - doesn't matter for static files)
   - **Operating System**: Linux
   - **Region**: Choose closest to your users
   - **Pricing Plan**: Choose B1 Basic or higher (F1 Free tier is available but limited)
6. Click "Review + create" then "Create"

## Step 4: Deploy Files to Azure

### Method 1: Using Azure Portal (Easiest)

For **App Service**:

1. Go to your Web App in Azure Portal
2. In the left menu, find "Advanced Tools" under "Development Tools"
3. Click "Go" to open Kudu
4. Click "Debug console" → "CMD"
5. Navigate to `/home/site/wwwroot`
6. Drag and drop these files:
   - `main.crx`
   - `updates.xml`
   - `web.config` (for IIS) or `.htaccess` (for Apache)

For **Static Web Apps**:

1. In Azure Portal, go to your Static Web App
2. Click on "Browse" to see your site
3. Use the Azure CLI or GitHub Actions for deployment (see Method 2)

### Method 2: Using Azure CLI

Install Azure CLI: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli

```bash
# Login to Azure
az login

# For App Service deployment
az webapp deployment source config-zip --resource-group YOUR_RESOURCE_GROUP --name YOUR_WEB_APP_NAME --src deploy.zip

# First, create deploy.zip with main.crx, updates.xml, and web.config
```

### Method 3: Using Git Deployment

1. In Azure Portal, go to your Web App
2. Navigate to "Deployment Center"
3. Choose "Local Git"
4. Get the Git URL
5. Push your files to this repository

```bash
git init
git add main.crx updates.xml web.config
git commit -m "Initial deployment"
git remote add azure YOUR_AZURE_GIT_URL
git push azure main
```

## Step 5: Configure Web Server Headers

### For Azure App Service (Linux with nginx):

Create a file named `nginx.conf` or add to existing configuration:

```nginx
location ~ \.crx$ {
    add_header Content-Type application/x-chrome-extension;
    add_header Access-Control-Allow-Origin *;
}
```

### For Azure App Service (Windows/IIS):

The `web.config` file we created should work automatically.

## Step 6: Repackage Extension with Update URL

1. Update `manifest.json` with your Azure URL in the `update_url` field
2. Repackage the extension:
   - Go to `chrome://extensions/`
   - Click "Pack extension"
   - Select your extension folder
   - Use the existing `.pem` key if you have one, or create new
   - **SAVE THE .pem FILE SECURELY** - you'll need it for future updates
3. Upload the new `main.crx` to Azure

## Step 7: Test Your Deployment

1. Visit your Azure URL in a browser:
   - App Service: `https://YOUR_APP_NAME.azurewebsites.net/main.crx`
   - Static Web App: `https://YOUR_STATIC_APP.azurestatikapps.net/main.crx`

2. Check that the file downloads with the correct MIME type:
   - Open browser Developer Tools (F12)
   - Go to Network tab
   - Download the .crx file
   - Check Response Headers: should show `Content-Type: application/x-chrome-extension`

3. Test the updates.xml:
   - Visit: `https://YOUR_DOMAIN/updates.xml`
   - Should show the XML content with your extension ID and version

## Step 8: Install Extension on Linux

On a Linux machine with Chrome:

1. Download the extension: `wget https://YOUR_DOMAIN.azurewebsites.net/main.crx`
2. Open Chrome: `chrome://extensions/`
3. Drag and drop the `main.crx` file onto the extensions page
4. Chrome will install it

## Step 9: Updating Your Extension

When you need to release a new version:

1. Update the version in `manifest.json` (e.g., from "1.0.0" to "1.0.1")
2. Repackage the extension using the **same .pem key**:
   ```
   chrome --pack-extension=/path/to/extension --pack-extension-key=/path/to/extension.pem
   ```
3. Update `updates.xml` with the new version number
4. Upload both new `main.crx` and `updates.xml` to Azure
5. Chrome will auto-update within a few hours (or force update from chrome://extensions/)

## Troubleshooting

### Extension won't install
- Check that Content-Type header is set to `application/x-chrome-extension`
- Verify the .crx file is not corrupted
- Make sure HTTPS is enabled (Azure provides this by default)

### Updates not working
- Verify `update_url` in manifest.json matches your Azure URL
- Check that `updates.xml` is accessible and has correct appid
- Ensure version number in updates.xml is higher than installed version
- Try forcing update: chrome://extensions/ → "Update" button

### CORS errors
- Add Access-Control-Allow-Origin header in web.config or server config
- Check Azure App Service CORS settings in Portal

## Security Best Practices

1. **Protect your .pem file**: Never commit it to Git or share it
2. **Use HTTPS only**: Azure provides this by default
3. **Consider authentication**: For private extensions, add authentication to your Azure App
4. **Monitor access**: Use Azure Application Insights to track downloads

## Cost Considerations

- **Static Web Apps**: Free tier includes 100 GB bandwidth/month
- **App Service B1**: ~$13/month, includes 10 GB storage
- **App Service F1 (Free)**: 1 GB bandwidth/day, 165 MB storage

For a Chrome extension with minimal traffic, either free tier should suffice.

## Additional Resources

- [Chrome Extension Distribution Guide](https://developer.chrome.com/docs/extensions/how-to/distribute/host-on-linux)
- [Azure Static Web Apps Documentation](https://docs.microsoft.com/en-us/azure/static-web-apps/)
- [Azure App Service Documentation](https://docs.microsoft.com/en-us/azure/app-service/)
