# Bing Rewards Blocker Extension

A browser extension for Microsoft Edge and Google Chrome that blocks the Bing rewards API to hide the rewards icon and count from Bing.com.

## What it does

This extension blocks the URL `https://www.bing.com/rewardsapp/widgetassets` which is used by Bing to display the rewards icon and points counter. When this API is blocked, the rewards widget will no longer appear on Bing search pages.

## Technical Details

- Uses Manifest V3 (the latest extension standard)
- Uses `declarativeNetRequest` API for efficient URL blocking
- Blocks all resource types from the rewards API endpoint
- No background scripts needed, making it lightweight and efficient

Self-Hosted Extension is in https://calm-river-0b3f57803.2.azurestaticapps.net/

