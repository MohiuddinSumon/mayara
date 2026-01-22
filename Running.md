# Testing MAYARA Without a Physical Radar
 
This document explains how to test MAYARA in replay mode using captured radar network traffic (pcap files).
 
## Overview
 
MAYARA can be tested without physical radar hardware by using **replay mode** with pre-recorded network packet captures (pcap files). This allows developers and testers to work with the software even when no radar is available.
 
## What is Replay Mode?
 
Replay mode (`--replay` flag) tells MAYARA to:
- Process incoming network packets as if they came from a real radar
- Disable command sending (read-only mode - you can't control the radar)
- Process packets from any network interface, including loopback
- Update timestamps to current time for each spoke
- Draw visual indicators (circles at last two pixels of each spoke)
 
## Why tcpreplay?
 
**tcpreplay** is a tool that replays network traffic from pcap files back onto a network interface. Its role in testing MAYARA is:
 
1. **Simulates Radar Network Traffic**: Radar data is transmitted over UDP/multicast. tcpreplay takes the captured packets and injects them back onto the network interface.
 
2. **Enables Testing Without Hardware**: Instead of needing a physical radar connected to your network, you can replay previously captured radar traffic.
 
3. **Reproducible Testing**: Using the same pcap file ensures consistent, reproducible test scenarios.
 
4. **Development Workflow**: Developers can test code changes against known good radar data without needing access to physical equipment.
 
### How tcpreplay Works
 
```
[PCAP File] → [tcpreplay] → [Network Interface (lo)] → [MAYARA listening on interface]
```
 
tcpreplay reads the pcap file and sends each packet to the specified network interface at the same timing (or adjusted timing) as when it was originally captured. MAYARA, listening on that interface, receives these packets and processes them as if they came from a real radar.
 
## Quick Start Guide
 
### Prerequisites
 
1. **Build MAYARA**:
   ```bash
   cargo build --release
   ```
 
2. **Install tcpreplay**:
   ```bash
   sudo apt-get install -y tcpreplay
   ```
   (On other systems: `yum install tcpreplay` or `brew install tcpreplay`)
 
### Step-by-Step Testing
 
1. **Start MAYARA in replay mode**:
   ```bash
   ./target/release/mayara-server --replay --brand navico -i lo -p 6502
   ```
   Options explained:
   - `--replay`: Enable replay mode
   - `--brand navico`: Specify radar brand (navico, furuno, raymarine, garmin)
   - `-i lo`: Listen on loopback interface (localhost)
   - `-p 6502`: Web server port (default is 6502)
 
2. **In a separate terminal, replay the sample pcap file**:
   ```bash
   sudo tcpreplay -i lo /home/bs01532/mayara/demo/samples/halo_and_0183.pcap
   ```
   For continuous replay (loops forever):
   ```bash
   sudo tcpreplay -q -T select -l 0 -i lo /home/bs01532/mayara/demo/samples/halo_and_0183.pcap
   ```
 
3. **Access the web interface**:
   - Main page: http://localhost:6502
   - Radar viewer: http://localhost:6502/viewer.html
   - Control interface: http://localhost:6502/control.html
   - API endpoint: http://localhost:6502/v1/api/radars
 
4. **Verify radar detection**:
   ```bash
   curl http://localhost:6502/v1/api/radars
   ```
   Should return JSON with detected radar(s).
 
## What We Just Did
 
Here's what was executed to get MAYARA running in replay mode:
 
1. **Built the project**:
   ```bash
   cargo build --release
   ```
   This compiled the Rust code into an optimized binary at `target/release/mayara-server`.
 
2. **Started MAYARA server**:
   ```bash
   ./target/release/mayara-server --replay --brand navico -i lo -p 6502
   ```
   The server is now running in the background, listening on the loopback interface for Navico radar packets.
 
3. **Verified it's running**:
   - Checked the API endpoint: `curl http://localhost:6502/v1/api/radars`
   - Checked the interfaces: `curl http://localhost:6502/v1/api/interfaces`
   - Confirmed it's listening for Navico radars on loopback interface
 
4. **Next step** (requires tcpreplay):
   Replay the sample pcap file to inject radar data into the loopback interface so MAYARA can process it.
 
## Using Your Own Pcap Files
 
If you have access to a radar network, you can capture your own pcap files:
 
1. **Capture radar traffic**:
   ```bash
   sudo tcpdump -i <interface> -w my_radar_capture.pcap
   ```
   Replace `<interface>` with the network interface connected to the radar (e.g., `eth0`, `wlan0`).
 
2. **Replay your capture**:
   ```bash
   ./target/release/mayara-server --replay --brand <brand> -i lo
   sudo tcpreplay -i lo my_radar_capture.pcap
   ```
 
## Troubleshooting
 
- **No radar detected**: Make sure tcpreplay is running and sending packets to the same interface (`-i lo`) that MAYARA is listening on.
- **Permission denied**: tcpreplay requires root/sudo to inject packets into network interfaces.
- **Wrong brand**: Make sure the `--brand` flag matches the radar brand in the pcap file.
- **Port already in use**: Use `-p <different_port>` to specify a different port.
 
## Alternative: Docker Demo
 
The project includes a Docker-based demo setup that handles everything automatically:
 
```bash
cd demo
./build.sh
docker run --name mayara-demo -p 3000-3001:3000-3001 keesverruijt/mayara:latest
```
 
This starts MAYARA, Signal K server, and automatically replays the pcap file.