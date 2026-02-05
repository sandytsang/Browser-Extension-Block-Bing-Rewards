# GitHub Actions Deployment Setup for Azure Static Web Apps

This guide will help you set up automatic deployment to Azure Static Web Apps using GitHub Actions.

## Step 1: Create Azure Static Web App

### Option A: Using Azure Portal

1. Go to [Azure Portal](https://portal.azure.com)
2. Click "Create a resource"
3. Search for "Static Web App" and select it
4. Click "Create"
5. Fill in the details:
   - **Subscription**: Select your subscription
   - **Resource Group**: Select `rg-browser-extension`
   - **Name**: `bing-rewards-blocker` (or your preferred name)
   - **Plan type**: Free
   - **Region**: Choose the closest region (e.g., East US 2, West Europe)
   - **Deployment details**:
     - **Source**: GitHub
     - **Organization**: sandytsang
     - **Repository**: ExtensionBlockBingRewards
     - **Branch**: main
   - **Build Details**:
     - **Build Presets**: Custom
     - **App location**: /
     - **Api location**: (leave empty)
     - **Output location**: (leave empty)
6. Click "Review + create"
7. Click "Create"

**Important**: After creation, Azure will automatically:
- Create a GitHub Actions workflow file in your repository
- Add the `AZURE_STATIC_WEB_APPS_API_TOKEN` secret to your GitHub repository

### Option B: Using Azure CLI

```powershell
# Login to Azure
az login

# Create the Static Web App
az staticwebapp create `
  --name bing-rewards-blocker `
  --resource-group rg-browser-extension `
  --location "East US 2" `
  --source https://github.com/sandytsang/ExtensionBlockBingRewards `
  --branch main `
  --app-location "/" `
  --login-with-github
```

## Step 2: Get Deployment Token (Manual Setup - Only if not using Portal creation)

If you created the Static Web App through CLI or need to set up manually:

1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to your Static Web App resource
3. In the left menu, click on "Settings" → "Overview"
4. Click "Manage deployment token"
5. Copy the deployment token

## Step 3: Add GitHub Secret (Manual Setup - Only if needed)

If the secret wasn't added automatically:

1. Go to your GitHub repository: https://github.com/sandytsang/ExtensionBlockBingRewards
2. Click on "Settings" tab
3. In the left sidebar, click "Secrets and variables" → "Actions"
4. Click "New repository secret"
5. Name: `AZURE_STATIC_WEB_APPS_API_TOKEN`
6. Value: Paste the deployment token from Step 2
7. Click "Add secret"

## Step 4: Verify GitHub Actions Workflow

The workflow file `.github/workflows/azure-static-web-apps-deploy.yml` has been created. It will:

- Trigger on push to `main` branch
- Trigger on pull requests
- Can be manually triggered from GitHub Actions tab
- Deploy your extension files to Azure Static Web Apps

## Step 5: Update Azure URL in Configuration Files

After your Static Web App is created, you'll get a URL like:
`https://nice-forest-0a1b2c3d4.1.azurestaticapps.net`

Update the following files with your actual URL:

1. **manifest.json**:
```json
"update_url": "https://YOUR-STATIC-WEB-APP-URL.azurestaticapps.net/updates.xml"
```

2. **updates.xml**:
```xml
<updatecheck codebase='https://YOUR-STATIC-WEB-APP-URL.azurestaticapps.net/main.crx' version='1.0.0' />
```

Or if you set up a custom domain, use that instead.

## Step 6: Deploy

### Automatic Deployment

Simply push to the main branch:

```powershell
git add .
git commit -m "Set up Azure Static Web Apps deployment"
git push origin main
```

GitHub Actions will automatically deploy your extension!

### Manual Trigger

1. Go to your GitHub repository
2. Click on "Actions" tab
3. Select "Azure Static Web Apps CI/CD" workflow
4. Click "Run workflow"
5. Select branch "main"
6. Click "Run workflow"

## Step 7: Verify Deployment

1. Wait for the GitHub Action to complete (check the Actions tab)
2. Once complete, visit your Static Web App URL
3. Test the following URLs:
   - `https://YOUR-APP-URL.azurestaticapps.net/` (should show index.html)
   - `https://YOUR-APP-URL.azurestaticapps.net/main.crx` (should download the extension)
   - `https://YOUR-APP-URL.azurestaticapps.net/updates.xml` (should show XML)

4. Check the .crx file headers:
   - Open browser DevTools (F12)
   - Go to Network tab
   - Download the .crx file
   - Check Response Headers: should include `Content-Type: application/x-chrome-extension`

## Step 8: Custom Domain (Optional)

To use a custom domain like `extension.yourdomain.com`:

1. In Azure Portal, go to your Static Web App
2. Click "Custom domains" in the left menu
3. Click "Add"
4. Choose "Other" (if using external DNS)
5. Enter your domain name
6. Add the required DNS records to your domain registrar:
   - Type: CNAME
   - Name: extension (or your subdomain)
   - Value: [your-static-web-app-url].azurestaticapps.net
7. Click "Add"
8. Wait for DNS propagation (can take up to 48 hours)
9. Update manifest.json and updates.xml with your custom domain

## Monitoring and Troubleshooting

### View Deployment Logs

1. Go to GitHub repository → Actions tab
2. Click on the latest workflow run
3. Click on "Build and Deploy Job"
4. Expand steps to see detailed logs

### View Azure Logs

1. Go to Azure Portal
2. Navigate to your Static Web App
3. Click "Monitoring" → "Application Insights" (if enabled)

### Common Issues

**Issue**: Workflow fails with authentication error
**Solution**: Verify `AZURE_STATIC_WEB_APPS_API_TOKEN` secret is set correctly

**Issue**: .crx file not installing
**Solution**: Check Content-Type header is set to `application/x-chrome-extension` in staticwebapp.config.json

**Issue**: Updates not working
**Solution**: Verify update_url in manifest.json matches your Azure URL exactly

**Issue**: 404 on deployed files
**Solution**: Ensure files (main.crx, updates.xml) are in the root directory and committed to git

## Updating Your Extension

To release a new version:

1. Update version in `manifest.json` (e.g., "1.0.0" → "1.0.1")
2. Repackage the extension with the same .pem key
3. Replace `main.crx` with the new version
4. Update version in `updates.xml`
5. Commit and push:
```powershell
git add manifest.json main.crx updates.xml
git commit -m "Release version 1.0.1"
git push origin main
```
6. GitHub Actions will automatically deploy the update
7. Chrome will auto-update the extension within a few hours

## Cost

Azure Static Web Apps Free tier includes:
- 100 GB bandwidth per month
- 2 custom domains
- Free SSL certificates
- Free for personal projects

Perfect for hosting a Chrome extension!

## Next Steps

1. Set up a custom domain (optional)
2. Enable Application Insights for monitoring
3. Set up branch previews for testing
4. Add more extensions to the same Static Web App
