# Troubleshooting Guide - Zyxel Router App for Homey

This guide helps you fix common issues with the Zyxel Router app (Zyxel NR5307) for Homey Pro.

## Table of Contents
- [Pairing fails: "Could not connect to the router"](#pairing-fails-could-not-connect-to-the-router)
- [Device shows "Cannot connect to router"](#device-shows-cannot-connect-to-router)
- [Logged out of the router's web interface](#logged-out-of-the-routers-web-interface)
- [Connected devices](#connected-devices)
- [Signal, 5G and bands](#signal-5g-and-bands)
- [Data usage and bandwidth](#data-usage-and-bandwidth)
- [Latency](#latency)
- [WiFi shows "Off"](#wifi-shows-off)
- [Reboot](#reboot)
- [Reading the logs](#reading-the-logs)
- [Reporting issues](#reporting-issues)

---

## Pairing fails: "Could not connect to the router"

**Message**: "Could not connect to the router. Please check the IP address and credentials."

The app logs in to the router before it adds the device. This message means that login failed.

### Step 1: Check the IP address
1. In a browser on the same network, open `https://192.168.8.1` (the NR5307 default; use your router's IP if you changed it)
2. Your browser warns about the certificate. That is normal: the router uses a self-signed certificate. Accept the warning and continue.
3. If you see the login page, the IP address is correct

**Finding your router's IP**: it is the *default gateway* in your computer's or phone's network settings, or check the label on the router.

### Step 2: Check the username and password
1. Log in to the router's web interface with the same details
2. The username is usually `admin` (lowercase). The password is the admin password you set, or the one printed on the router's label.
3. Username and password are case-sensitive; watch out for extra spaces when pasting

If the browser login works but pairing doesn't, see [Reading the logs](#reading-the-logs).

### Step 3: Same network
- Homey and the router must be on the same network. Homey's IP should start with the same three numbers, e.g. `192.168.8.x`.
- A guest network or VLAN with client isolation can block Homey from reaching the router

---

## Device shows "Cannot connect to router"

The device is marked unavailable when the router doesn't answer the status poll. It becomes available again by itself once the router answers.

1. **Is the router reachable?** Open `https://[router-ip]` in a browser
2. **Changed the password or IP?** Use **Repair** in the device settings and enter the new details. You don't need to remove the device, and your Flows keep working.
3. **Router just restarted?** Give it 2–3 minutes after power-on
4. **Router power-cycle**: unplug it for 30 seconds, plug it back in and wait 2–3 minutes

---

## Logged out of the router's web interface

The router allows **one admin session at a time**. While the app is polling, logging in to the web interface can log the app out, and the app's next login can then log *you* out.

- The app logs back in automatically; nothing needs fixing
- For a longer session in the web interface, temporarily disable the app (Homey → Apps → Zyxel Router → Disable), or raise the poll intervals

---

## Connected devices

### Number of devices looks wrong
The router also keeps devices that were connected earlier. The app only counts devices the router marks as **active**. On the settings page, inactive devices are greyed out and marked *(Offline)*.

### List does not update
1. Check **Devices - Poll Interval** in the device settings (default 120 seconds)
2. Open the app's settings page and click **↻ Refresh**
3. Restart the app: Homey → Apps → Zyxel Router → Restart

### "Device connected / disconnected" Flows fire late
These triggers fire on the next devices poll after a change, so with the default interval that can take up to 2 minutes. Triggers don't fire on the very first poll after the app starts.

---

## Signal, 5G and bands

### Values are empty or not changing
1. Check that the router has an active cellular connection (router web interface)
2. Check **Signal - Poll Interval** (default 60 seconds, minimum 10)
3. In bridge mode, or when the router uses an Ethernet WAN instead of cellular, there is no signal data

### 5G RSRP / 5G SINR are empty
These are only filled while a 5G NR carrier is active. On LTE only, they are empty, and **Network technology** shows `LTE`.

### Reading the values
| Value | Meaning |
|---|---|
| Signal strength | LTE RSRP as %: −120 dBm = 0 %, −80 dBm = 100 % |
| Network technology | e.g. `LTE`, `NR5G+` |
| Active bands | LTE carriers / 5G bands, e.g. `B3+B7 / N78+N1` |

---

## Data usage and bandwidth

### Monthly usage restarted at 0 after updating to 1.2.0
That is expected, and it happens only once. Before 1.2.0 the app measured the LAN interface instead of the internet (WAN) traffic, so the old count wasn't correct. From 1.2.0 the monthly counter restarts once from the current WAN reading.

### Monthly usage differs from my provider
- The app can only count while it runs. Traffic while Homey or the app is off is partly missed.
- Set **Billing cycle reset day** to the day your provider's billing period starts
- Use your provider's figures as the reference for billing

### Bandwidth shows 0 or a low value
Bandwidth is the **average** over the **Bandwidth - Poll Interval** (default 60 seconds), not a speed test. Short bursts are averaged out. After a router reboot, one sample is skipped.

---

## Latency

Latency is measured **from Homey**, with a TCP connection to port 443 of the **Latency target** (default `1.1.1.1`). ICMP ping is not used because Odido blocks it.

- If the target doesn't respond, the app tries `1.1.1.1` and `8.8.8.8`
- If none respond, the value is not updated. Check Homey's internet connection.
- Devices paired before 1.2.0 keep their old target `8.8.8.8`. You can change it in the device settings.

---

## WiFi shows "Off"

The WiFi fields show **Off** when the router doesn't broadcast that network. A guest network counts as off while the main WiFi on the same band is off, even if the guest network itself is enabled in the router. This is common when you use separate access points (e.g. Ubiquiti) and the router's own WiFi is switched off.

---

## Reboot

After the **Reboot** action, the device shows "Router is rebooting…" and polling pauses. After 60 seconds the app tries to log in every 30 seconds (up to 10 times), then polling resumes automatically.

---

## Reading the logs

Open the app's logs via the Homey Developer Tools (https://tools.developer.homey.app) or with `homey app run` during development.

### Normal operation
```
Signal updated — RSRP: -103 dBm, strength: 43%, NR5G+ [B3+B7 / N78+N1], 5G RSRP: -100 dBm
Connected devices: 28 active of 42 known
Router status updated
Bandwidth — ↓ 4.3 Mbps  ↑ 0.0 Mbps (over 62s)
Latency (TCP 1.1.1.1:443): 26 ms
```

### Common errors
| Log message | Meaning |
|---|---|
| `Invalid username or password` | The router rejected the login: check the credentials, then use Repair |
| `Cannot connect to router at …` | Router not reachable over HTTPS: check the IP and the network |
| `Update failed [signal]: …` | One poll failed. The app retries on the next interval. |
| `Failed to get router status: …` | Status poll failed. The device is marked unavailable until the router answers again. |
| `Request failed after 2 authentication retries` | The router keeps refusing the session. Often caused by someone being logged in to the web interface. |
| `TCP connect to … failed, trying next target…` | Latency target unreachable. The next target is tried. |

### Test the router from a computer
```bash
curl -k https://192.168.8.1/getBasicInformation
```
This needs no login. It returns the model and firmware version, e.g. `"ModelName":"NR5307"`. If it doesn't respond, the problem is the network, not the app.

---

## Reporting issues

If none of the above helps, [open an issue on GitHub](https://github.com/Rob2011/Homey.Zyxel.Router/issues) with:

1. **Router**: model and firmware version (from `getBasicInformation` above or the web interface), and whether it uses cellular or Ethernet WAN
2. **App version** (Homey → Apps → Zyxel Router)
3. **Network**: router IP, Homey IP
4. **Error**: the exact message, when it happens (pairing, after hours, after a reboot), and a screenshot if possible
5. **Logs**: the relevant lines from the app log. **Remove passwords, IMEI and other personal data first.**
6. **What you tried** and what happened

---

## Quick reference

### Default settings
| Setting | Default |
|---|---|
| IP address | `192.168.8.1` |
| Username | `admin` |
| Signal - Poll Interval | 60 s |
| Devices - Poll Interval | 120 s |
| Router Status - Poll Interval | 300 s |
| Bandwidth - Poll Interval | 60 s |
| Latency target | `1.1.1.1` |
| Billing cycle reset day | 1 |
| Monthly data limit (GB) | 0 (off) |

### Quick fixes
| Problem | Quick fix |
|---|---|
| Cannot pair | Check IP/credentials in a browser first |
| Password changed | Device settings → Repair |
| Logged out of web interface | Expected (one admin session); disable the app temporarily |
| Devices not updating | Check the poll interval, click Refresh on the settings page |
| 5G values empty | No 5G carrier active at that moment |
| Monthly usage reset after update | One-time switch to WAN counters in 1.2.0 |

---

**Last updated**: version 1.2.4
