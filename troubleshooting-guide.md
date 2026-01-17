# Troubleshooting Guide - Zyxel NR5307 Homey App

This guide helps you resolve common issues with the Zyxel NR5307 router app for Homey Pro.

## Table of Contents
- [Connection Failed Error](#connection-failed-error)
- [Cannot Connect to Router](#cannot-connect-to-router)
- [Authentication Issues](#authentication-issues)
- [Device List Problems](#device-list-problems)
- [Signal Quality Not Updating](#signal-quality-not-updating)
- [Advanced Debugging](#advanced-debugging)

---

## Connection Failed Error

**Error Message**: "Connection failed: Failed to connect to router. Please check IP address and credentials."

### Step 1: Verify Router IP Address

1. Open a web browser on any device connected to your network
2. Try to access your router at: `https://192.168.1.1` (or your router's IP)
3. If you see a login page, the IP is correct
4. If you get a timeout or "cannot reach" error, the IP is wrong

**Find your router's IP:**
- Check the label on the router (usually on the back/bottom)
- Common Zyxel router IPs: `192.168.1.1`, `192.168.0.1`, `10.0.0.1`
- Check your computer's network settings to see the default gateway

### Step 2: Test Router Connection from Homey's Network

1. Make sure Homey Pro is on the same network as the router
2. Check if there are multiple networks (e.g., guest WiFi) that might isolate devices
3. Ensure no firewall is blocking Homey from accessing the router

### Step 3: Verify Credentials

1. Open your router's web interface: `https://[router-ip]`
2. Try logging in with the credentials you entered in the Homey app
3. Default credentials are usually:
   - Username: `admin`
   - Password: `1234` or printed on router label

**Important**: 
- Username and password are **case-sensitive**
- Make sure there are no extra spaces
- If you changed the password, use the new one

### Step 4: Check HTTPS Access

The router requires HTTPS (secure connection). If you see certificate warnings in your browser, that's normal - just accept and continue.

---

## Cannot Connect to Router

### Symptom
App cannot reach the router at all, timeout errors.

### Solutions

#### Check Network Connectivity

1. **Ping test from another device:**
   ```bash
   ping 192.168.1.1
   ```
   If this fails, there's a network problem.

2. **Check Homey's network:**
   - Open Homey settings → Network
   - Verify Homey is connected to WiFi/Ethernet
   - Check if Homey can access the internet

3. **Router location:**
   - Is the router between Homey and the internet?
   - Check if both are on the same subnet (e.g., 192.168.1.x)

#### Firewall Issues

1. Check if your router has firewall rules blocking local access
2. Some routers have "management access" settings - ensure LAN access is enabled
3. Check if MAC filtering is enabled (whitelist Homey's MAC address)

#### Router Issues

1. **Reboot the router:**
   - Unplug power for 30 seconds
   - Plug back in and wait 2-3 minutes for full boot

2. **Check router status lights:**
   - Power: Should be solid
   - Internet: Should be solid or blinking
   - If lights are unusual, check router manual

---

## Authentication Issues

### "Invalid username or password"

1. **Verify in browser first:**
   - Open `https://[router-ip]` in a browser
   - Log in with the same credentials
   - If browser login fails, your credentials are wrong

2. **Common issues:**
   - Default username is `admin` (lowercase)
   - Password might have been changed during setup
   - Check for special characters that might be copied incorrectly

3. **Reset router password (last resort):**
   - Use the reset button on the router (hold 10-15 seconds)
   - **WARNING**: This resets ALL settings, not just password
   - You'll need to reconfigure the router

### "Session validation failed"

This means login worked but the session isn't valid:

1. **Try again:** Sometimes transient network issues cause this
2. **Clear and re-add device:** Remove the device from Homey and add it again
3. **Router firmware:** Check if router firmware needs updating

---

## Device List Problems

### No devices showing (0 active devices)

#### Check Logs

1. Open Homey Developer Tools (if available) or check app logs
2. Look for messages like:
   ```
   Device: MacBook Air, Active field value: true, active: true
   Device list retrieved: 26 devices (26 active)
   ```

3. If you see `Active field value: undefined`, the router isn't returning active status

#### Possible Causes

1. **Router API difference:**
   - Different firmware versions may use different field names
   - Check GitHub issues for your specific router model/firmware

2. **All devices offline:**
   - Verify devices are actually connected (check router web interface)
   - Try connecting a device and refreshing the list

3. **API endpoint issue:**
   - The app might need updating for your router's firmware version
   - Report the issue on GitHub with your firmware version

### Devices shown but all appear offline (greyed out)

1. **Check API response format:**
   - Open an issue on GitHub with logs
   - Include your router's firmware version

2. **Temporary workaround:**
   - Use the router's web interface to view devices
   - Export device list from settings page for reference

### Device list not updating

1. **Check polling interval:**
   - Go to device settings
   - Verify "Device Poll Interval" is not too high (60 seconds recommended)

2. **Manual refresh:**
   - Open device settings page
   - Click "Refresh" button

3. **Restart app:**
   - Go to Homey Settings → Apps
   - Find "Zyxel NR5307"
   - Click restart

---

## Signal Quality Not Updating

### Symptom
RSRP, RSRQ, RSSI, SINR values not changing or showing 0.

### Solutions

1. **Check cellular connection:**
   - Verify router has an active cellular connection
   - Check router web interface for signal strength

2. **Polling interval:**
   - Go to device settings
   - Check "Signal Poll Interval" (30 seconds recommended)
   - Set to a lower value if needed (minimum: 10 seconds)

3. **Router mode:**
   - If router is in bridge mode or using WAN instead of cellular, signal data won't be available
   - Verify router is using cellular/5G connection

---

## Advanced Debugging

### Enable Detailed Logging

1. Open Homey CLI or Developer Tools
2. Look for log entries with these symbols:
   - `✓` = Success
   - `→` = Request sent
   - `←` = Response received
   - `⚠` = Warning

### Common Log Messages

#### Successful Connection
```
→ Sending login request...
← Login response status: 200
← Received 1 cookie(s)
✓ Session cookie found
✓ Authentication successful!
```

#### Connection Problem
```
⚠ Could not get basic info (trying alternate method)
✗ Cannot reach router at 192.168.1.1
```

#### Authentication Problem
```
← Login response: ZCFG_INVALID_ARGUMENTS
✗ Invalid username or password
```

### Test Connection Manually

From a computer on the same network:

```bash
# Test if router is reachable
curl -k https://192.168.1.1/getBasicInformation

# Test login (replace with your credentials)
curl -k -X POST https://192.168.1.1/UserLogin \
  -H "Content-Type: application/json" \
  -d '{"Input_Account":"admin","Input_Passwd":"[base64-password]","currLang":"en","RememberPassword":0,"SHA512_password":false}'
```

### Check Router Firmware

1. Log into router web interface
2. Go to Maintenance → Firmware
3. Note your firmware version
4. Check if an update is available
5. Report firmware version when creating GitHub issues

### Network Diagnostics

1. **Check Homey's IP address:**
   - Settings → Network
   - Note IP and subnet (e.g., 192.168.1.50/24)

2. **Verify same subnet:**
   - Router: 192.168.1.1
   - Homey: 192.168.1.x (should match first 3 numbers)
   - If different, check network configuration

3. **Test from Homey's location:**
   - Use a laptop/phone in the same location as Homey
   - Try accessing router web interface
   - If that fails, it's a signal/range issue

---

## Reporting Issues

If none of the above solutions work, please report the issue on GitHub with:

### Information to Include

1. **Router details:**
   - Model: Zyxel NR5307 (or variant)
   - Firmware version: (from router web interface)
   - Connection type: Cellular/Ethernet/Bridge mode

2. **Network setup:**
   - Router IP address: (e.g., 192.168.1.1)
   - Homey IP address: (from Homey settings)
   - Same network: Yes/No

3. **Error details:**
   - Exact error message
   - When it occurs (during pairing, after X hours, etc.)
   - Screenshot if possible

4. **Logs:**
   - Relevant log entries from Homey
   - Any console output with the ✓ → ← symbols

5. **What you've tried:**
   - List troubleshooting steps already attempted
   - Results of each attempt

### Creating a GitHub Issue

1. Go to: https://github.com/[your-repo]/issues
2. Click "New Issue"
3. Use template: "Connection Problem" or "Bug Report"
4. Include all information from above
5. Be as detailed as possible

---

## Quick Reference

### Default Settings
- Router IP: `192.168.1.1`
- Username: `admin`
- Password: (on router label, often `1234`)
- Signal Poll: 30 seconds
- Device Poll: 60 seconds
- Status Poll: 300 seconds

### Common Issues Quick Fixes

| Problem | Quick Fix |
|---------|-----------|
| Cannot connect | Verify IP, check network |
| Wrong password | Try default: `admin` / `1234` |
| Devices not updating | Check polling interval |
| Signal at 0 | Verify cellular connection active |
| Connection drops | Increase polling intervals |

### Support Resources

- **GitHub Issues**: Report bugs and get help
- **Homey Community Forum**: Ask questions, share experiences
- **Router Manual**: For router-specific settings
- **Homey Developer Tools**: For detailed logging

---

## Prevention Tips

1. **Use static IP for router:** Prevents IP changes
2. **Document your credentials:** Store securely
3. **Keep firmware updated:** Both router and Homey
4. **Monitor logs periodically:** Catch issues early
5. **Test after changes:** Verify still working after network changes

---

**Last Updated**: Version 1.0.0

For more help, visit the GitHub repository or Homey Community Forum.