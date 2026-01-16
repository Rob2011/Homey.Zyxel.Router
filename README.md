# Zyxel NR5307 Router App for Homey Pro

A Homey Pro app for monitoring and managing your Zyxel NR5307 5G router. 
View connected devices, monitor signal quality, and control your network directly from Homey.

![Zyxel Router](https://img.shields.io/badge/Zyxel-NR5307-blue)
![Homey](https://img.shields.io/badge/Homey-Pro-orange)
![License](https://img.shields.io/badge/license-MIT-green)

## Features

### Device Monitoring
- **Connected Devices List** - View all devices connected to your router (active and inactive)
- **Real-time Updates** - Automatic polling for device status and signal quality
- **Device Details** - See IP addresses, MAC addresses, and connection types (Ethernet/WiFi)
- **Export Device List** - Download a text file with all connected devices

### Signal Quality Monitoring
- Monitor cellular signal metrics:
  - RSRP (Reference Signal Received Power)
  - RSRQ (Reference Signal Received Quality)
  - RSSI (Received Signal Strength Indicator)
  - SINR (Signal-to-Interference-plus-Noise Ratio)

### Router Status
- **Uptime Tracking** - Monitor how long your router has been running
- **WAN/LAN IP Addresses** - View external and internal IP addresses
- **Signal Strength** - Current cellular connection strength
- **Data Traffic** - Monitor data sent and received

<p align="center"><img width="400px" alt="Device list" class="recess" src="./images/device list.png" /></p>


### Flow Cards

#### Triggers
- **Signal Quality Changed** - Triggered when signal metrics update
- **Device Connected** - Triggered when a new device joins the network
- **Device Disconnected** - Triggered when a device leaves the network

#### Conditions
- **Signal Quality Above Threshold** - Check if RSRP is above a specified value
- **Device Count Above** - Check if the number of connected devices exceeds a threshold

#### Actions
- **Reboot Router** - Restart your router remotely
- **Enable WiFi** - Turn on WiFi
- **Disable WiFi** - Turn off WiFi
- **Block Device** - Block a device by MAC address
- **Unblock Device** - Unblock a previously blocked device

## Installation

- Install the test version: https://homey.app/nl-nl/app/zyxel.router/Zyxel-Router/test/

### From Homey App Store
1. Open the Homey app
2. Go to "More" → "Apps"
3. Search for "Zyxel NR5307"
4. Click "Install"

### Manual Installation (Development)
1. Clone this repository
2. Run `npm install`
3. Run `homey app install` (requires Homey CLI)

## Setup

1. Add the app from the Homey app store
2. Go to "Devices" → "Add Device" → "Zyxel NR5307"
3. Enter your router details:
   - **Router IP Address** (default: `192.168.1.1`)
   - **Username** (default: `admin`)
   - **Password** (your router password)
4. Configure polling intervals:
   - **Signal Poll Interval** - How often to check signal quality (default: 30 seconds)
   - **Device Poll Interval** - How often to check connected devices (default: 60 seconds)
   - **Status Poll Interval** - How often to check router status (default: 300 seconds)

## Device Settings Page

The app includes a custom settings page showing all connected devices with:
- Device name and icon
- IP address and MAC address
- Connection type (Ethernet/WiFi)
- Active/Offline status (inactive devices are greyed out)
- Export functionality to download the device list

Access the settings page from the device settings in the Homey app.

## Requirements

- Homey Pro
- Zyxel NR5307 5G Router alsao known in NL as "Klik&Klaar" from Odido
- Router must be accessible on your local network
- HTTPS access to router (self-signed certificate is handled automatically)

## Configuration

### Router Settings
The app connects to your router using HTTPS. Default settings:
- IP Address: `192.168.1.1`   (your routers IP Address)
- Username: `admin`
- Password: (your router password)

### Polling Intervals
Adjust how often the app checks your router:
- **Signal Quality**: 30 seconds (recommended)
- **Connected Devices**: 60 seconds (recommended)
- **Router Status**: 300 seconds (recommended)

Lower intervals provide more real-time data but increase network traffic.

## Troubleshooting

### Cannot connect to router
- Verify the router IP address is correct
- Check username and password
- Ensure the router is accessible on your network
- Try accessing `https://[router-ip]` in a browser

### Signal quality not updating
- Verify the signal poll interval is set correctly
- Check if cellular connection is active
- Review logs for authentication errors

## API Endpoints

The app uses the Zyxel router's DAL (Data Access Layer) API:
- `/cgi-bin/DAL?oid=status` - Router status
- `/cgi-bin/DAL?oid=lanhosts` - Connected devices
- `/cgi-bin/DAL?oid=cellwan_status` - Cellular signal quality
- `/cgi-bin/DAL?oid=TrafficStatus` - Data traffic statistics

## Development

### Structure
```
zyxel-nr5307/
├── app.js                 # Main app file
├── drivers/
│   └── zyxel-nr5307/
│       ├── device.js      # Device driver
│       ├── driver.js      # Driver logic
│       └── assets/
│           └── index.html # Settings page
├── zyxel-api.js          # Router API client
└── api.js                # Homey API endpoints
```

## Contributing

Contributions or suggestions are welcome! Please:
- Submit a pull request

## Known Issues

- Export device list under development
- Some routers may have different API endpoints
- Self-signed certificate warnings (handled automatically)

## Changelog

### Version 1.0.1
- Initial release
- Connected device monitoring
- Signal quality tracking
- Router status monitoring
- Flow card support
- Custom settings page with export functionality

## License

MIT License - see LICENSE file for details

## Author

Created for the Homey Pro community

## Support

For issues, questions, or feature requests, please open an issue on GitHub.

## Acknowledgments

- Zyxel for the NR5307 router
- Athom for the Homey platform

---

**Note**: This is an unofficial app and is not affiliated with or endorsed by Zyxel.
