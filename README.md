# Secure Raspberry Pi ADS-B Flight Feeder Build

## Project Mission
**Why this guide exists:**<br> Existing online documentation for setting up an ADS-B receiver often skips critical steps. This guide provides a complete, step-by-step walkthrough to fill those gaps, making data contribution accessible to enthusiasts of all technical backgrounds and inspiring them to explore hands-on technology projects.

---

**What this project does:**<br>This project details the deployment and configuration of an edge-computing ADS-B (Automatic Dependent Surveillance-Broadcast) receiver. The system utilizes a Raspberry Pi running Linux to capture raw 1090 MHz radio frequency telemetry from aircraft transponders and securely feed the decoded data to FlightRadar24.

Note: This documentation was refined using AI-assisted editing to improve clarity and presentation, while maintaining my original analysis and findings.


---

## Security Advisory: Network Segmentation
Before deploying any internet-facing edge device or automated data feeder on your home network, it is highly recommended to isolate the device to protect your primary systems.
* **Recommendation:** Look into your router's documentation regarding VLANs (Virtual Local Area Networks) or Guest Network Isolation / AP Isolation.
* **Objective:** Ensure the Raspberry Pi has outbound internet access to feed data but is strictly prohibited from communicating with other local devices on your primary home network.

---

## Hardware Requirements

* **Single-Board Computer (SBC):** Raspberry Pi Model 3B+ or newer
* **Storage:** 8 GB MicroSD card (minimum)<br> \*Be sure to purchase from a reputable dealer - counterfit cards are everywhere.
* **Power:** 5.1V / 3A Power Supply
* **Receiver:** ADS-B USB DVB-T Dongle with matching antenna
* **Enclosure:** Case with access to all physical ports and passive heatsinks (recommended)

---

## Deployment Steps
### Image Provisioning & Headless Config
1. Assemble the Raspberry Pi into its enclosure and apply the heatsinks.

2. Download the official Pi24 image from https://www.flightradar24.com/build-your-own and extract the archive.

3. Download and install https://etcher.balena.io/.<br><br><small><small>(Technical Note on Flashing: Standard file transfers (drag-and-drop) only copy surface-level files. Tools like balenaEtcher are required to write the underlying Master Boot Record (MBR), establish the correct Linux partitions, and clone the disk image sector-by-sector so the Raspberry Pi can successfully initialize the bootloader.)</small></small><br>
4. Flash the extracted Pi24 image to the MicroSD card using balenaEtcher.

5. Optional Wi-Fi Configuration: If using a wireless connection, navigate to the boot directory of the flashed SD card, locate wpa_supplicant.conf, update the ssid and psk fields with your network credentials, and save the file.

## Node Hardening & Initial Boot

6. Insert the MicroSD card into the Raspberry Pi. Connect the ADS-B USB dongle, a monitor, a keyboard, and power on the device.

7. Allow the boot sequence to complete, note the assigned IP address displayed on the screen, and press Ctrl + Alt + F2 to open the Linux TTY terminal.

8. Authenticate using the default credentials (pi : raspberry) and immediately launch the configuration utility:
   ```bash
   sudo raspi-config
   ```
9. Critical Hardening Step: Change the default user password immediately by running:
   ```bash
   passwd
   ```
10. Enable the secure shell daemon to allow remote administration, and verify its active status:
    ```bash
    sudo systemctl enable --now ssh
    ```
    ```bash
    sudo systemctl status ssh
    ```
## Remote Administration & Service Installation

11. Disconnect the peripherals from the Pi. Move to your primary workstation, open a terminal, and establish an SSH connection:
    ```bash
    ssh pi@<YOUR_PI_IP_ADDRESS>
    ```
12. Execute the FlightRadar24 installation and configuration script:
    ```bash
    wget -qO- [https://fr24.com/install.sh](https://fr24.com/install.sh) | sudo bash -s
    ```
    
13. Follow the interactive configuration wizard prompts.

\*Note: Select NO on step 5.1 (unless actively sharing raw data with a secondary, separate tracking network) and select NO on step 5.2.

## Verification & Triage

14. Upon completion, verify the feeder daemon status:
    ```bash
    fr24feed-status
    ```
    
* **If the initial link state shows unknown or failed, restart the system service to force synchronization:
  ```bash
  sudo systemctl restart fr24feed
  ```
    
* **Wait approximately 15 seconds and re-verify the service status to confirm the UDP link is established and data packets are processing:
  ```bash
  fr24feed-status
  ```
    
* **To view your unique sharing key or configuration profile at any time, inspect the initialization file:
  ```bash
  cat /etc/fr24feed.ini
  ```
    
15. Terminate the remote session: 
  ```bash
  exit
  ```

## Verification Links & Local Diagnostics

Local Web Dashboard: http://<YOUR_PI_IP_ADDRESS>:8754 (Monitor receiver performance and status locally)

Official Statistics & Account Claiming: https://www.flightradar24.com/share-your-data

You won't have to notify FlightRadar24 that you did this or anything, your account will automatically be upgraded and show that you are a contributor. Enjoy the free membership!
