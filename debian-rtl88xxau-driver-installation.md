# rtl88xxau Driver Installation Guide for Debian

## Introduction
The rtl8812au driver is a wireless LAN driver for devices using the Realtek RTL8812AU chipset. It provides support for Wi-Fi connectivity and various features such as monitor mode, TX power adjustment, LED control, and more.

This guide will walk you through the installation process of the rtl8812au driver on Debian, ensuring that you can use your Realtek RTL8812AU chipset-based wireless device with all its capabilities.

This installation guide is not limited to the rtl8812au chipset but can be applied to any Realtek RTL88xxAU chipset-based wireless device. By following the steps outlined in this guide, you can install the rtl88xxau driver on Debian and utilize the full capabilities of your device.

## Prerequisites

Before starting the installation, make sure you have the following prerequisites:

- A Debian system (version X or higher).
- An active internet connection.
- Basic knowledge of using the terminal and running commands as root (using sudo).

## Manual Installation Steps

On Debian, the `sudo apt install realtek-rtl88xxau-dkms` command will not work as it is specific to Kali Linux. Instead, we need to manually clone the driver from GitHub. Here are the steps:

#### Update the package information

```bash
sudo apt update
```

#### Install dkms and git
```bash
sudo apt install dkms git
```

#### Install Build Dependencies
```bash
sudo apt install build-essential libelf-dev linux-headers-$(uname -r)
```

#### Download the Driver files using git
```bash
git clone https://github.com/aircrack-ng/rtl8812au.git
```

#### Navigate to the Downloaded directory
```bash
cd rtl88*
```

#### Install the driver using DKMS:
```bash
sudo make dkms_install
```

- if the installation is aborted , check existing dkms modules and uninstall previously installed driver

---
#### Uninstall Existing Driver

- check the module name and version using the command `sudo dkms status`.
```bash
$ dkms status  
```
```text
8812au, 5.6.4.2_35491.20191025, 5.10.63+, armv6l: installed  
rtl8188fu, 1.0, 5.10.63+, armv6l: installed.``` 
```

- here module name is 8812au and module version is 5.6.4.2_35491.20191025.

- use `sudo dkms remove <module>/<module-version>`.
```bash
$ sudo dkms remove 8812au/5.6.4.2_35491.20191025 --all
```
```text
Deleting module version: 5.6.4.2_35491.20191025 completely from the DKMS tree.
Done.
```

- delete this file using `sudo rm -rf /var/lib/dkms/8812au/`.

---
- now try to perform "Install the driver using DKMS"

The rtl8812au driver should now be installed on your system. You can check if it's loaded by running:

```bash
$ sudo modprobe 88XXau
```

These commands will compile the driver and install it using DKMS (Dynamic Kernel Module Support). Finally, the modprobe command loads the driver into the kernel.

---
You can now configure additional features such as monitor mode, TX power, LED control, etc.

For example, to set up monitor mode:

```bash
$ sudo airmon-ng check kill
$ sudo ip link set wlan0 down
$ sudo iw dev wlan0 set type monitor
$ sudo ip link set wlan0 up
```

Optionally, you can configure LED control by creating a configuration file:

```bash
$ sudo nano /etc/modprobe.d/8812au.conf
```

Add the following line to the file and save it:

```text
options 88XXau rtw_led_ctrl=0
```

Alternatively, you can control the LED dynamically by writing to `/proc/net/rtl8812au/$(your interface name)/led_ctrl`.

That's it! The rtl8812au driver should now be installed on your Debian system, and you can configure it according to your requirements.

---
## Troubleshooting

### Issue: Key Rejected by Service

If you encounter the following error message when trying to insert the 88XXau module using modprobe:
```bash
$ sudo modprobe 88XXau
```
```text
modprobe: ERROR: could not insert '88XXau': Key was rejected by service
```

This error usually occurs when Secure Boot is enabled on your system, which requires signed kernel modules. To resolve this issue, you have a few options:

#### Disable Secure Boot (Recommended):

- Restart your system and access the BIOS/UEFI settings.
- Look for the Secure Boot option and disable it.
- Save the changes and restart your system.
- Try loading the module again using `sudo modprobe 88XXau`.

#### Sign the module manually (Advanced):

- Generate your own key and certificate pair using the openssl command.
- Sign the module manually using the sign-file script.
- Load the module using modprobe.
- This method requires some technical knowledge and is more complex. You can find detailed instructions in the Linux kernel documentation for signing modules.

---
##### Note: 

- Disabling Secure Boot or manually signing modules may have security implications. Make sure you understand the risks and implications before proceeding.

- Try disabling Secure Boot first, as it is the recommended approach. If that doesn't work or if you require Secure Boot enabled, you can explore the manual signing method.

- These instructions cover the manual installation process for the rtl8812au driver on Debian, as well as additional steps for testing packet injection capabilities and switching to monitor mode.

---
## Testing and Additional Configurations

After installing the rtl8812au driver, you may want to perform some additional tests and configurations. Here are the steps:

#### Switching to Monitor Mode

To change the wireless interface mode to monitor mode, use the following commands:

```bash
sudo airmon-ng check kill
sudo iwconfig wlan0 mode monitor
```

These commands prepare the adapter for monitoring wireless networks.

Verify that the wireless interface mode has been successfully changed to monitor mode by executing the following command:

```bash
iwconfig
```

This step ensures that the adapter is ready for monitoring wireless network activity.

#### Test packet injection capabilities using the following command

To test the packet injection capabilities of the wireless adapter, use the command:

```bash
sudo aireplay-ng --test wlan0
```

This command helps you determine if the adapter supports packet injection, which is often required for certain security testing or network analysis tasks.

- Check if the adapter supports the 5GHz band using the `iwconfig` command and the `airodump-ng` command with the `--band a` option.

```bash
iwconfig
sudo airodump-ng --band a wlan0
```

This step helps you identify if the adapter can operate on the 5GHz frequency.

- To switch back to the 2.4GHz band, use the `airodump-ng` command with the `--band bg` option.

Use the following command to switch the wireless adapter back to the 2.4GHz band:

```bash
sudo airodump-ng --band bg wlan0
```

This command is useful if you want to monitor or analyze networks operating on the 2.4GHz frequency.

#### Check if the adapter supports Soft-AP

To check if the wireless adapter supports Soft-AP (Access Point), use the following command:

```bash
sudo airbase-ng -a 00:01:02:03:04:05 --essid "fake wife" -c 11 wlan0
```

---
