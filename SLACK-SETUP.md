# Slack Integration Setup Guide

This guide explains how to configure Slack integration for NAC-Tap to enable automatic PCAP and PCredz output uploads, as well as heartbeat monitoring.

## Overview

The Slack integration provides:
- **Heartbeat Monitoring**: Status updates every 15 seconds showing appliance status, capture state, and PCAP size
- **File Uploads**: Automatic upload of PCAP files and PCredz output every 60 seconds
- **Remote Access**: Monitor and retrieve capture data from Slack without direct device access

## Prerequisites

- A Slack workspace with admin permissions (or ability to create apps)
- Internet connectivity from the NAC-Tap appliance
- WiFi AP connection configured (see Upload tab in WebUI)

## Step 1: Create a Slack App

1. Go to [https://api.slack.com/apps](https://api.slack.com/apps)
2. Click **"Create New App"**
3. Select **"From scratch"**
4. Enter app details:
   - **App Name**: `NAC-Tap` (or any name you prefer)
   - **Workspace**: Select your workspace
5. Click **"Create App"**

## Step 2: Create Incoming Webhook (for Heartbeats and Messages)

1. In your app settings, go to **"Incoming Webhooks"** in the left sidebar
2. Toggle **"Activate Incoming Webhooks"** to **On**
3. Scroll down and click **"Add New Webhook to Workspace"**
4. Select the channel where you want to receive messages (e.g., `#nac-tap`)
5. Click **"Allow"**
6. **Copy the Webhook URL** - it looks like:
   ```
   https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX
   ```
7. Save this URL securely - you'll need it for configuration

## Step 3: Enable Bot Token Scopes (for File Uploads)

1. In your app settings, go to **"OAuth & Permissions"** in the left sidebar
2. Scroll down to **"Scopes"** section
3. Under **"Bot Token Scopes"**, click **"Add an OAuth Scope"**
4. Add the following scopes:
   - `files:write` - Upload files to Slack
   - `chat:write` - Send messages (if not already present)
5. Scroll to the top of the page
6. Click **"Install to Workspace"**
7. Review permissions and click **"Allow"**
8. **Copy the Bot User OAuth Token** - it starts with `xoxb-`
   ```
   xoxb-1234567890-1234567890123-AbCdEfGhIjKlMnOpQrStUvWx
   ```
9. Save this token securely - you'll need it for configuration

## Step 4: Get Channel ID or Name

You can use either:
- **Channel name** (e.g., `#nac-tap`) - recommended, easier to use
- **Channel ID** (e.g., `C1234567890`) - more specific

### Option A: Use Channel Name (Easier)

Simply use the channel name with the `#` prefix:
- Example: `#nac-tap`
- Ensure the channel exists and the bot has access

### Option B: Get Channel ID

1. In Slack, right-click on the channel name
2. Select **"View channel details"** (or similar)
3. Scroll to the bottom - the Channel ID is listed there
4. It will look like: `C1234567890`

## Step 5: Configure NAC-Tap WebUI

1. Open the NAC-Tap WebUI in your browser (default: `http://10.200.66.1:8080`)
2. Navigate to the **"Upload"** tab
3. Connect to a WiFi AP (if not already connected):
   - Select WLAN interface (e.g., `wlan0`)
   - Click **"Scan for APs"**
   - Select an AP from the list
   - Enter WiFi password if required
   - Click **"Connect"**
   - Wait for connection confirmation
   - Click **"Test Internet"** to verify connectivity

4. Configure Slack settings:
   - **Webhook URL**: Paste the webhook URL from Step 2
   - **Bot Token**: Paste the bot token from Step 3
   - **Channel**: Enter channel name (e.g., `#nac-tap`) or channel ID
   
5. Test the configuration:
   - Click **"Test Webhook"** - should show "Webhook test successful!"
   - Click **"Test Token"** - should show "Token test successful!"
   - Click **"Test Integration"** - should show "Integration test successful!"

6. Configure upload options:
   - **Upload PCAP files**: Check to upload PCAP captures
   - **Upload PCredz output**: Check to upload loot.json and raw.txt files
   - You can enable one or both options

7. Save configuration:
   - Check **"Auto-connect WiFi on startup"** if you want automatic WiFi connection after reboot
   - Check **"Auto-enable upload on startup"** if you want automatic uploads after reboot
   - Click **"Save Configuration"** - this saves all settings to persist across reboots

8. Enable auto-upload:
   - Click **"Enable Upload"** to start automatic uploads and heartbeats
   - You should see confirmation: "Auto-upload enabled"

## Step 6: Verify Integration

### Check Heartbeat Messages

Within 15 seconds, you should receive a heartbeat message in your Slack channel:
```
NAC-Tap Heartbeat
Appliance: nac-tap-001
Status: online
Last Poll: 2025-11-08 14:32:15
Capture Active: yes
PCAP Size: 2.3 MB
```

### Check File Uploads

Within 60 seconds, if capture is active and files exist:
- PCAP file (if enabled) will be uploaded to Slack
- loot.json and raw.txt files (if enabled) will be uploaded to Slack
- A summary message will be posted with file links

### Test Manual Upload

1. Click **"Upload Now"** button in the Upload tab
2. Check Slack channel for uploaded files

## Configuration Details

### Heartbeat Frequency
- Default: Every 15 seconds
- Configurable in code: `HEARTBEAT_INTERVAL` in CONFIG

### Upload Frequency
- Default: Every 60 seconds
- Configurable in code: `UPLOAD_INTERVAL` in CONFIG

### Upload Options
- **PCAP Only**: Uploads only PCAP capture files
- **PCredz Only**: Uploads only loot.json and raw PCredz output
- **Both**: Uploads all files (default)

### Configuration Persistence

All settings are saved to:
```
/var/log/nac-captures/upload-config.json
```

This file:
- Persists across reboots
- Contains encrypted WiFi password (base64)
- Has 600 permissions (owner read/write only)
- Is automatically loaded on startup if auto-connect/auto-upload is enabled

## Troubleshooting

### Webhook Test Fails

1. **Check webhook URL format**:
   - Must start with `https://hooks.slack.com/services/`
   - Should have three segments separated by `/`

2. **Verify webhook is active**:
   - Go to Slack app settings > Incoming Webhooks
   - Ensure webhook is enabled and not revoked

3. **Check internet connectivity**:
   - Ensure WiFi is connected
   - Test internet: Click "Test Internet" button
   - Verify ping and DNS both show OK

### Token Test Fails

1. **Check token format**:
   - Should start with `xoxb-` (bot token)
   - Should be long (50+ characters)

2. **Verify scopes**:
   - Go to Slack app settings > OAuth & Permissions
   - Ensure `files:write` and `chat:write` scopes are added
   - Reinstall app if scopes were added after initial install

3. **Check app installation**:
   - Ensure app is installed to your workspace
   - Verify bot user exists in your workspace

### Integration Test Fails

1. **Verify channel access**:
   - Ensure bot is invited to the channel (if private channel)
   - Check channel name/ID is correct

2. **Test each component separately**:
   - Test webhook first
   - Test token second
   - If both pass individually but integration fails, check channel access

### No Heartbeat Messages

1. **Check upload is enabled**:
   - Ensure "Enable Upload" was clicked
   - Check upload status shows "Enabled: Yes"

2. **Verify WiFi connection**:
   - Heartbeats only send when WiFi is connected and internet is available
   - Check "Current Connection" box shows connected status

3. **Check logs**:
   - View status tab logs for error messages
   - Look for "Heartbeat failed" messages

### Files Not Uploading

1. **Check upload options**:
   - Verify PCAP or PCredz checkboxes are checked
   - Files only upload if the corresponding option is enabled

2. **Verify files exist**:
   - PCAP files are created when capture is active
   - PCredz output is created after running analysis

3. **Check internet connectivity**:
   - Uploads skip if internet is not available
   - Verify internet test passes

4. **Check file sizes**:
   - Slack has a 1GB file size limit
   - Large PCAP files may fail to upload

### Configuration Not Persisting

1. **Check file permissions**:
   ```bash
   ls -l /var/log/nac-captures/upload-config.json
   ```
   Should show `-rw-------` (600 permissions)

2. **Verify directory exists**:
   ```bash
   ls -ld /var/log/nac-captures/
   ```
   Directory should exist and be writable

3. **Check save confirmation**:
   - Ensure "Configuration saved!" message appeared
   - Check "Configuration loaded" appears on page load

## Security Best Practices

1. **Protect Webhook URL**:
   - Anyone with the webhook URL can post to your channel
   - Don't share webhook URL publicly
   - Rotate webhook if compromised

2. **Protect Bot Token**:
   - Bot token provides full file upload and message access
   - Store securely (config file has 600 permissions)
   - Rotate token if compromised (revoke and regenerate in Slack)

3. **Channel Access**:
   - Use a private channel for sensitive data
   - Only invite necessary team members
   - Monitor channel for unauthorized access

4. **File Privacy**:
   - PCAP files contain network traffic - may include sensitive data
   - PCredz output contains extracted credentials
   - Ensure only authorized personnel have channel access
   - Consider using Slack Enterprise Grid for advanced security

5. **Network Security**:
   - Use secure WiFi (WPA2/WPA3) for appliance connection
   - Encrypt configuration file (already implemented with base64 obfuscation)
   - Limit access to appliance management interface

## API Reference

### Endpoints Used

- **POST** `/api/slack/test/webhook` - Test webhook URL
- **POST** `/api/slack/test/token` - Test bot token
- **POST** `/api/slack/test/integration` - Test full integration
- **POST** `/api/slack/configure` - Configure Slack settings
- **POST** `/api/upload/enable` - Enable/disable auto-upload
- **POST** `/api/upload/trigger` - Manually trigger upload
- **GET** `/api/upload/status` - Get upload status and history
- **POST** `/api/config/save` - Save configuration
- **GET** `/api/config/load` - Load saved configuration

### Slack API Endpoints

- **POST** `https://hooks.slack.com/services/...` - Incoming webhook
- **POST** `https://slack.com/api/files.upload` - File upload
- **POST** `https://slack.com/api/auth.test` - Token validation

## Example Slack Messages

### Heartbeat Message
```
NAC-Tap Heartbeat
Appliance: nac-tap-001
Status: online
Last Poll: 2025-11-08 14:32:15
Capture Active: yes
PCAP Size: 2.3 MB
```

### Upload Summary Message
```
NAC-Tap Upload Complete
Appliance: nac-tap-001
Files uploaded:
- PCAP: capture-2025-11-08.pcap (2.3 MB)
- Loot: loot.json (45 KB)
- Raw: pcredz_output.txt (12 KB)
```

## Support

For issues or questions:
1. Check troubleshooting section above
2. Review NAC-Tap logs: `/var/log/auto-nac-bridge.log`
3. Check Slack app settings and permissions
4. Verify internet connectivity and firewall rules

## Notes

- Heartbeats and uploads only work when WiFi is connected and internet is available
- Files are uploaded to the Slack channel, not as direct messages
- Large PCAP files may take time to upload depending on network speed
- Slack has rate limits - excessive uploads may be throttled
- Configuration survives reboots if saved properly

