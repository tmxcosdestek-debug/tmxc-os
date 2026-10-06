# TMXC OS - Hardware Installation Guide

## ⚠️ Security Disclaimer

**IMPORTANT:** Installing TMXC OS on physical hardware involves modifying device firmware, unlocking bootloaders, and flashing custom operating system images. These actions may:

- Void your device manufacturer warranty
- Potentially brick your device if performed incorrectly
- Expose your device to security risks if proper precautions aren't taken
- Result in permanent data loss

**Proceed at your own risk.** The TMXC OS development team is not responsible for any damage to your device or loss of data. Always backup your data before proceeding.

## Prerequisites

### Hardware Requirements
- TMXC OS-compatible device (see [DEVICE_COMPATIBILITY.md](../../DEVICE_COMPATIBILITY.md))
- USB data cable (for connection to computer)
- Computer with internet connection
- Minimum 50% battery charge on target device

### Software Requirements
- TMXC OS activation key (obtain from tmxc.os.destek@gmail.com)
- Platform tools (ADB/Fastboot) for your computer
- TMXC OS device image (.img or .bin file)
- Device-specific bootloader unlock tool (varies by manufacturer)

## Installation Process

### Step 1: Obtain Activation Key

Before installing TMXC OS, you must obtain a valid activation key:

1. Send an email to **tmxc.os.destek@gmail.com**
2. Include your device model and serial number
3. Wait for activation key delivery (typically 24-48 hours)
4. Keep your activation key secure - you'll need it during first boot

### Step 2: Prepare Your Computer

**Windows:**
```powershell
# Download platform-tools from Android Developer website
# Extract to C:\platform-tools
# Add to system PATH
```

**Linux:**
```bash
sudo apt-get install android-tools-adb android-tools-fastboot
```

**macOS:**
```bash
brew install android-platform-tools
```

### Step 3: Enable Developer Options

1. Go to **Settings** → **About Phone**
2. Tap **Build Number** 7 times to enable Developer Options
3. Go back to **Settings** → **Developer Options**
4. Enable **USB Debugging**
5. Enable **OEM Unlocking** (if available)

### Step 4: Unlock Bootloader

**⚠️ WARNING:** This will wipe all data on your device.

#### For Most Android Devices:

```bash
# Reboot to bootloader
adb reboot bootloader

# Unlock bootloader (command varies by manufacturer)
fastboot oem unlock
# OR
fastboot flashing unlock
```

#### Device-Specific Instructions:

**Samsung Devices:**
- Download Odin tool
- Download Samsung bootloader unlock package
- Follow manufacturer-specific unlock process

**Xiaomi Devices:**
- Request unlock permission from Mi Unlock website
- Use Mi Flash Unlock tool
- Follow on-screen instructions

**Google Pixel:**
- Enable OEM unlock in Developer Options
- Use `fastboot flashing unlock`

**OnePlus:**
- Enable Advanced Reboot in Developer Options
- Use `fastboot oem unlock`

### Step 5: Enter Fastboot Mode

```bash
# From powered off state
# Hold Volume Down + Power button simultaneously
# OR use ADB
adb reboot bootloader
```

Verify connection:
```bash
fastboot devices
```

### Step 6: Flash TMXC OS Image

```bash
# Flash the TMXC OS image to appropriate partition
fastboot flash boot tmxc_os_boot.img
fastboot flash system tmxc_os_system.img
fastboot flash vendor tmxc_os_vendor.img

# OR flash combined image
fastboot flash boot tmxc_os_combined.img
```

### Step 7: Flash Recovery Image (Optional)

```bash
fastboot flash recovery tmxc_os_recovery.img
```

### Step 8: Reboot to System

```bash
fastboot reboot
```

### Step 9: First Boot Configuration

1. **TMXC OS will boot** (first boot may take 5-10 minutes)
2. **Language Selection** - Choose your preferred language
3. **Network Setup** - Connect to Wi-Fi or mobile network
4. **Activation** - Enter your activation key when prompted
5. **Device Setup** - Complete initial configuration

## Troubleshooting

### Device Won't Boot

**If device is stuck in bootloop:**
```bash
# Reboot to recovery
adb reboot recovery

# Wipe data/factory reset
# Reboot to bootloader
fastboot flash boot tmxc_os_boot.img
```

### Fastboot Commands Not Recognized

- Ensure platform-tools are in your PATH
- Try using full path to fastboot executable
- Check USB cable is data cable, not charging-only

### Bootloader Unlock Failed

- Verify your device supports bootloader unlocking
- Check if carrier restrictions apply
- Contact manufacturer for unlock code if required

### Activation Key Invalid

- Verify you entered the key correctly
- Contact tmxc.os.destek@gmail.com if key appears invalid
- Ensure your device model matches the one registered

## Post-Installation

### Verify Installation

```bash
# Check TMXC OS version
adb shell tmxc_version

# Check system status
adb shell tmxc_status
```

### Security Setup

1. Enable biometric authentication
2. Configure encryption vaults
3. Set up ghost mode if desired
4. Configure neural firewall settings

### Update System

```bash
# Check for updates
adb shell tmxc_update check

# Apply updates
adb shell tmxc_update install
```

## Recovery & Restoration

### Restoring Original Firmware

If you need to restore your device to stock firmware:

1. Download stock firmware for your device
2. Reboot to fastboot mode
3. Flash stock images:
```bash
fastboot flash boot stock_boot.img
fastboot flash system stock_system.img
fastboot flash vendor stock_vendor.img
fastboot flash recovery stock_recovery.img
```

### Emergency Recovery

If device is bricked:
- Use device-specific unbrick tools
- Contact TMXC OS support
- Consider professional repair service

## Additional Resources

- [Device Compatibility List](../../DEVICE_COMPATIBILITY.md)
- [Security Documentation](../security/activation_protocol.md)
- [Troubleshooting Guide](troubleshooting.md)
- [Community Forum](https://community.tmx-os.org)

## Support

For installation issues:
- **Email:** tmxc.os.destek@gmail.com
- **Documentation:** [docs.tmx-os.org](https://docs.tmx-os.org)
- **Community:** [community.tmx-os.org](https://community.tmx-os.org)

---

**Last Updated:** 2026-07-15  
**TMXC OS Version:** 1.0.0  
**Guide Version:** 1.0
