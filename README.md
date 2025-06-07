# **Raspberry Pi 5 Chromium Kiosk Mode Setup**

This guide provides a step-by-step walkthrough to set up a Raspberry Pi 5 to boot directly into a full-screen Chromium web browser, ideal for dedicated dashboard displays or digital signage.

## **What is needed for this project:**

-   Raspberry Pi 5
-   MicroSD Card (16GB or larger recommended)
-   Raspberry Pi OS (64-bit) with Desktop (latest version)
-   Raspberry Pi Imager
-   SSH Client (e.g., PuTTY)
-   Ethernet Cable (recommended for initial setup)
-   Monitor, Keyboard, Mouse (for initial setup and troubleshooting)
-   Your target URL (e.g., `http://192.168.0.101:19999`)

---

# **Setup the Raspberry Pi 5 for Kiosk Mode**

-   **Prepare the MicroSD Card:**
    -   Download the latest **Raspberry Pi OS (64-bit) with Desktop** image.
    -   Use **Raspberry Pi Imager** to flash the OS to your microSD card.
    -   In the Imager's **Advanced Options** (gear icon), enable **SSH** and set a username and password. This is crucial for remote access.
    -   Configure Wi-Fi settings if you plan to use wireless.
    -   Click "Write" to begin flashing.

-   **Initial Boot and Connect via SSH:**
    -   Insert the microSD card into the Raspberry Pi 5.
    -   Connect an Ethernet cable (or ensure Wi-Fi is configured) and power on the Pi.
    -   Find the Pi's IP address by checking your router's connected devices list. If needed, temporarily connect a monitor and keyboard to the Pi, log in, and run `ip a`.
    -   From your computer, use an SSH client (like PuTTY) to connect to the Pi's IP address (port 22).
    -   Log in using the username and password you set during flashing.

-   **Update System Packages:**
    ```bash
    sudo apt update
    sudo apt full-upgrade -y
    ```

-   **Configure Automatic Desktop Login:**
    -   Run the Raspberry Pi configuration tool:
        ```bash
        sudo raspi-config
        ```
    -   Navigate using arrow keys and Enter:
        -   Select **"1 System Options"**.
        -   Select **"S5 Boot / Auto Login"**.
        -   Select **"B2 Desktop GUI"**.
    -   Press Enter to confirm, then Escape to exit `raspi-config`. When prompted to reboot, select **"No"**.

-   **Create the Chromium Kiosk Systemd Service:**
    -   Create a new service file:
        ```bash
        sudo nano /etc/systemd/system/kiosk.service
        ```
    -   Paste the following content into the file.
        **Important:**
        -   Replace `geek` with your actual username where specified.
        -   Replace `http://192.168.0.101:19999` with your desired target URL.

        ```ini
        [Unit]
        Description=Chromium Kiosk
        After=graphical.target display-manager.service

        [Service]
        ExecStart=/usr/bin/chromium-browser --incognito --kiosk --disable-infobars --noerrdialogs --start-fullscreen http://192.168.0.101:19999
        WorkingDirectory=/home/geek
        StandardOutput=inherit
        StandardError=inherit
        Restart=always
        RestartSec=10
        User=geek
        Environment="DISPLAY=:0"
        Environment="XAUTHORITY=/home/geek/.Xauthority"

        [Install]
        WantedBy=graphical.target
        ```
    -   Save and exit: Press `Ctrl+O`, then `Enter`, then `Ctrl+X`.

-   **Enable Kiosk Service and Manage System Behavior:**
    -   Enable the `kiosk.service` to start automatically:
        ```bash
        sudo systemctl enable kiosk.service
        ```
    -   Mask the screen saver to prevent it from activating:
        ```bash
        sudo systemctl mask xscreensaver.service
        ```
    -   Disable automatic system updates for kiosk stability:
        ```bash
        sudo systemctl disable apt-daily.timer
        sudo systemctl disable apt-daily-upgrade.timer
        sudo systemctl stop apt-daily.timer
        sudo systemctl stop apt-daily-upgrade.timer
        ```

-   **Remove Conflicting Desktop Autostart (Recommended):**
    -   Edit the default desktop autostart file:
        ```bash
        sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
        ```
    -   Remove or comment out all existing lines in this file by adding a `#` at the beginning of each line. You can also leave the file empty.
    -   Save and exit: Press `Ctrl+O`, then `Enter`, then `Ctrl+X`.

-   **Final Reboot and Verification:**
    -   Reboot your Raspberry Pi:
        ```bash
        sudo reboot
        ```
    -   Observe the connected monitor. The Pi should boot, log in automatically, and then launch Chromium directly into full-screen kiosk mode, displaying your target webpage.

---

# **Customization Options**

## **Adjust Chromium Zoom Level**

To make text and content appear larger on the display:

1.  **SSH into your Pi.**
2.  **Edit your `kiosk.service` file:**
    ```bash
    sudo nano /etc/systemd/system/kiosk.service
    ```
3.  **Modify the `ExecStart` line** to include `--force-device-scale-factor=X.X`.
    -   **Example for 150% zoom:**
        ```bash
        ExecStart=/usr/bin/chromium-browser --incognito --kiosk --disable-infobars --noerrdialogs --start-fullscreen --force-device-scale-factor=1.5 http://192.168.0.101:19999
        ```
    -   Adjust `1.5` to your desired zoom level (e.g., `1.25` for 125%, `2.0` for 200%).
4.  **Save and exit.**
5.  **Reload systemd and restart the service:**
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart kiosk.service
    ```
6.  **Reboot** the Pi for the change to take effect.

## **Display Resolution Auto-Scaling**

The Raspberry Pi 5, when running Raspberry Pi OS Desktop, utilizes modern display drivers (Kernel Mode Setting or KMS). This provides excellent plug-and-play functionality:

-   **Automatic Detection:** The Pi is designed to automatically detect and use the optimal resolution of your connected display.
-   **No Manual Configuration:** In most cases, you will not need to manually configure display resolutions in system files. Simply connect your new display, and the Pi should adapt.

If this was helpful, consider subscribing to you YouTube channel at www.youtube.com/@GeekOfAllTrades
