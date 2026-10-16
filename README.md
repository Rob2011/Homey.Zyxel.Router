# Zyxel Router App for Homey Pro

Your 5G connection at a glance.

A Homey Pro app for the Zyxel NR5307 5G router (sold in the Netherlands by Odido as "Klik & Klaar"). Follow your 4G/5G signal, data usage and connected devices, and automate on them with Flows.

![Zyxel Router](https://img.shields.io/badge/Zyxel-NR5307-blue)
![Homey](https://img.shields.io/badge/Homey-Pro-orange)
![Version](https://img.shields.io/badge/version-1.2.4-lightgrey)
![License](https://img.shields.io/badge/license-MIT-green)

## Features

### Cellular signal (4G and 5G)
- **Signal strength** in % (based on LTE RSRP: −120 dBm = 0 %, −80 dBm = 100 %)
- **LTE metrics**: RSRP, RSRQ, RSSI and SINR
- **5G NR metrics**: 5G RSRP and 5G SINR when a 5G carrier is active
- **Network technology**, e.g. `LTE` or `NR5G+`
- **Active bands**, e.g. `B3+B7 / N78+N1` (LTE carriers incl. carrier aggregation / 5G NR bands)

### Data usage and performance
- **Download and upload bandwidth** (Mbps), measured on the WAN interface
- **Monthly data usage** with a configurable billing day and an optional data limit
- **Total data sent and received** since the router's last reboot
- **Latency** (ms), measured from Homey with a TCP handshake to port 443. ICMP ping is blocked on the Odido network, so a TCP connect is used instead.

### Router status
- Uptime, WAN and LAN IP address, WAN connection status
- Firewall level and firmware version
- WiFi network names and channels per band. A network shows **Off** when it is not broadcast. A guest network counts as off while the main WiFi on that band is off.

### Connected devices
- Number of active devices
- Device list on the app's settings page, with name, IP address, MAC address and connection type. Inactive devices are greyed out.
- Export the device list as a text file

<p align="center"><img width="400px" alt="Device list" class="recess" src="./images/device list.png" /></p>

## Flow cards

Homey shows the router above each card, so the titles don't repeat it.

### Triggers
| Card | Tokens |
|---|---|
| Signal strength changed | signal strength (%) |
| Network technology changed (e.g. LTE ↔ 5G) | technology, previous technology, active bands |
| Connection status changed | WAN status |
| Device connected | device name, MAC address |
| Device disconnected | device name, MAC address |
| Connected devices list updated | device count, device list, device details (JSON) |
| Monthly data limit exceeded | data used (GB), data limit (GB) |

Change triggers only fire on an actual change, not on every poll.

### Conditions
- **Is / is not connected via 5G**
- **Signal quality is / is not above** a threshold (dBm, LTE RSRP)
- **Latency is / is not below** a threshold (ms)

### Actions
- **Reboot**: restarts the router. The device shows as unavailable while the router restarts, and polling resumes automatically once it's back.

## Installation

Search for **Zyxel Router** in the Homey App Store, or install the test version:
https://homey.app/nl-nl/app/zyxel.router/Zyxel-Router/test/

## Setup

1. In the Homey app go to **Devices → Add device → Zyxel Router**
2. Enter the router details:
   - **IP address**: default `192.168.8.1`
   - **Username**: default `admin`
   - **Password**: your router's admin password
3. The app logs in to check the details before the device is added.

To change the IP address or password later, use **Repair** in the device settings. You don't need to remove the device.

## Device settings

| Setting | Default | Description |
|---|---|---|
| Signal poll interval | 60 s | Signal, technology and bands |
| Devices poll interval | 120 s | Connected devices and connect/disconnect triggers |
| Router status poll interval | 300 s | Uptime, IP addresses, WiFi, firewall; latency is measured on the same cadence |
| Bandwidth poll interval | 60 s | Bandwidth and data usage |
| Latency target | `1.1.1.1` | Host for the TCP latency measurement (falls back to 1.1.1.1 and 8.8.8.8) |
| Billing cycle reset day | 1 | Day of the month on which the monthly data usage resets |
| Monthly data limit (GB) | 0 | Triggers "Monthly data limit exceeded" once per cycle; 0 = off |

Shorter intervals give more up-to-date values but put more load on the router.

## Requirements

- Homey Pro (local platform)
- Zyxel NR5307. Other Zyxel NR models with similar firmware may work (e.g. NR7101, NR5103, NR5101, NR5309, NR5111, NR2301, NR7302), but they haven't been tested.
- The router must be reachable from Homey on the local network

## Security

- The app talks to the router over HTTPS. The router uses a self-signed certificate, which the app accepts.
- The app only falls back to plain HTTP if the router actively refuses HTTPS connections. Over HTTP the password would effectively travel unencrypted.
- The router allows one admin session at a time. If you log in to the router's web interface while the app is polling, the two can log each other out. The app logs back in automatically.

## Troubleshooting

### Cannot connect to router
- Check the IP address (default `192.168.8.1`), username and password
- Open `https://192.168.8.1` in a browser on the same network and accept the certificate warning
- Make sure Homey and the router are on the same network

### Values are not updating
- Check the poll intervals in the device settings
- Check that the cellular connection is up
- Look at the app's logs in the Homey Developer Tools

See the [Troubleshooting Guide](troubleshooting-guide.md) for more.

## How it works

The app uses the router's DAL (Data Access Layer) API:

| Endpoint | Used for |
|---|---|
| `POST /UserLogin` | Login (session cookie + CSRF session key) |
| `GET /getBasicInformation` | Firmware version (no login needed) |
| `GET /cgi-bin/DAL?oid=cellwan_status` | 4G/5G signal, technology, bands |
| `GET /cgi-bin/DAL?oid=lanhosts` | Connected devices |
| `GET /cgi-bin/DAL?oid=status` | Uptime, WiFi |
| `GET /cgi-bin/DAL?oid=cardpage_status` | WAN/LAN IP and WAN status |
| `GET /cgi-bin/DAL?oid=firewall` | Firewall level |
| `GET /cgi-bin/DAL?oid=Traffic_Status` | Traffic counters (WAN interfaces) |
| `POST /cgi-bin/Reboot?sessionkey=…` | Reboot |

## Development

```
├── app.js                  # App entry point
├── api.js                  # Web API for the settings page / widget
├── zyxel-api.js            # Router API client (login, DAL queries, reboot)
├── lib/helpers.js          # Pure parsing helpers (signal, bands, traffic, WiFi)
├── drivers/nr5307/
│   ├── device.js           # Polling, capabilities, Flow triggers
│   ├── driver.js           # Pairing, repair, Flow card listeners
│   ├── pair/ repair/       # Pairing and repair screens
├── settings/index.html     # Connected devices page
├── .homeycompose/          # Capabilities, Flow cards, app manifest
└── test/                   # Unit tests (npm test)
```

```bash
npm install
npm test
homey app run --remote
```

## Changelog

### 1.2.4
- Clearer app description and Flow card titles

### 1.2.3
- A guest WiFi network no longer shows as on while the main WiFi is off
- Added missing device image

### 1.2.0
- Data usage and bandwidth are now measured on the WAN interface (they used the LAN interface before). The monthly counter restarts once after the update.
- New: 5G RSRP and SINR, network technology, active bands, "Network technology changed" trigger, "Is connected via 5G" condition
- Flow cards fixed (device argument), change triggers only fire on real changes
- Faster settings page and widget, separate bandwidth interval, faster latency measurement
- No plain-HTTP login fallback anymore; smaller app package

### 1.1.0 and earlier
- Bug fixes, latency measurement, improved login, first release

## License

MIT License, see [LICENSE](LICENSE).

## Support

Having issues? Check the [Troubleshooting Guide](troubleshooting-guide.md) or [open an issue](https://github.com/Rob2011/Homey.Zyxel.Router/issues).

## Acknowledgments

- Zyxel for the NR5307 router
- Athom for the Homey platform

---

**Note**: This is an unofficial app. It is not affiliated with or endorsed by Zyxel.
