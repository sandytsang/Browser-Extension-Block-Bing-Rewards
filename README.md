# Bing Rewards Blocker Extension

A browser extension for Microsoft Edge and Google Chrome that blocks the Bing rewards API to hide the rewards icon and count from Bing.com.

## What it does

This extension blocks the URL `https://www.bing.com/rewardsapp/widgetassets` which is used by Bing to display the rewards icon and points counter. When this API is blocked, the rewards widget will no longer appear on Bing search pages.

## Installation

### Chrome

1. Open Chrome and navigate to `chrome://extensions/`
2. Enable "Developer mode" using the toggle in the top-right corner
3. Click "Load unpacked"
4. Select the folder containing this extension
5. The extension should now be installed and active

### Microsoft Edge

1. Open Edge and navigate to `edge://extensions/`
2. Enable "Developer mode" using the toggle in the bottom-left corner
3. Click "Load unpacked"
4. Select the folder containing this extension
5. The extension should now be installed and active

## Testing

1. Install the extension following the instructions above
2. Navigate to [https://www.bing.com](https://www.bing.com)
3. Open Developer Tools (F12)
4. Go to the Network tab
5. Search for requests to `rewardsapp/widgetassets`
6. You should see the request is blocked (or not appearing at all)
7. The rewards icon and counter should not be visible on the Bing homepage

## Files

- `manifest.json` - Extension configuration and metadata
- `rules.json` - Declarative rules for blocking the Bing rewards API
- `icons/` - Extension icons (16x16, 48x48, 128x128)

## Technical Details

- Uses Manifest V3 (the latest extension standard)
- Uses `declarativeNetRequest` API for efficient URL blocking
- Blocks all resource types from the rewards API endpoint
- No background scripts needed, making it lightweight and efficient

