# Post-Install Checklist — MSI X570 Hackintosh (macOS Sonoma)

Hardware profile inferred from EFI:

- **CPU**: AMD Ryzen
- **GPU**: AMD Radeon (`agdpmod=pikera`, WhateverGreen)
- **Audio**: AppleALC layout-id 5
- **WiFi**: Broadcom (AirportBrcmFixup) / Intel (AirportItlwm)
- **BT**: BrcmPatchRAM3 stack
- **Ethernet**: Realtek RTL8111 + SmallTreeIntel82576
- **USB**: Custom USBPorts + USBPower kexts
- **NVMe**: NVMeFix + SSDT-NVMe

---

### 0. EFI Deployment

1. Copy `.env.example` to `.env` and populate with your SMBIOS values:

   ```bash
   cp .env.example .env
   ```

   Edit `.env` and fill in your values:
   - `BOARD_SERIAL`: Board serial number (MLB)
   - `ROM`: MAC address in base64 format
   - `SERIAL`: System serial number
   - `UUID`: System UUID
   - `EFI`: Disk identifier (e.g., `/dev/disk0s1`)

2. Run the deploy script:

   ```bash
   sudo ./deploy
   ```

   This will:
   - Generate `config.plist` from the template by substituting your SMBIOS values
   - Mount the EFI partition
   - Backup the existing EFI (if present) to `~/Desktop/`
   - Copy the new EFI to the EFI partition
   - Unmount the EFI partition
   - Clean up the generated `config.plist`

## 1. Verify Hardware Functionality

### CPU & Power

- [ ] Run **Geekbench 6** or `stress-ng` — confirm all cores show up in Activity Monitor
- [ ] Confirm CPU temperature readings work: install **AMD Power Gadget** or **Stats** (uses AMDRyzenCPUPowerManagement + SMCAMDProcessor)
- [ ] Check `kern.bootargs` doesn't still have `debug=0x100` set (remove for daily use)

### GPU

- [ ] Open **About This Mac → System Report → Graphics** — confirm GPU name and VRAM are correct
- [ ] Run a Metal benchmark (Geekbench Metal or **GFXBench**) to verify acceleration
- [ ] Check GPU temperature via **Stats** or **RadeonSensor** menu bar widget (SMCRadeonGPU is loaded)
- [ ] Confirm **HDMI / DisplayPort** outputs work on all ports you plan to use
- [ ] Confirm display wake after sleep works (common AMD issue)

### Audio

- [ ] Test built-in audio output (3.5 mm jack and/or front panel)
- [ ] Test HDMI/DP audio output to a monitor or TV
- [ ] Test microphone input
- [ ] If audio is wrong, try other layout-ids for your codec with `alcid=X` in boot-args (currently `alcid=5`)

### Ethernet

- [ ] Confirm Realtek NIC shows up in **System Settings → Network**
- [ ] Run a speed test or `iperf3` to verify throughput
- [ ] If you have the Intel NIC (SmallTreeIntel82576), verify it appears too

### WiFi

- [ ] Two WiFi kexts are present (AirportBrcmFixup + AirportItlwm) — only one should be enabled for your card. Confirm the correct one is active in OpenCore config for your specific chip.
- [ ] Connect to a 2.4 GHz and 5 GHz network; test both bands
- [ ] Verify WiFi shows up as **AirPort** in menu bar (not generic WiFi) — required for AirDrop/Handoff

### Bluetooth

- [ ] Pair a Bluetooth device (headphones, keyboard, mouse)
- [ ] Check BT in **System Report → Bluetooth** — should show firmware version

### USB

- [ ] Test USB-A ports at USB 2.0 and USB 3.x speeds (use `AmorphousDiskMark` or `Blackmagic Disk Speed Test` on a USB SSD)
- [ ] Test USB-C port(s) if present
- [ ] Confirm no more than 15 USB ports are mapped (USBPorts.kext is present — good)
- [ ] Test front-panel USB ports specifically

### Storage / NVMe

- [ ] Verify NVMe drive shows correct name and capacity: `system_profiler SPNVMeDataType`
- [ ] Run **Blackmagic Disk Speed Test** — check read/write speeds are as expected
- [ ] Confirm TRIM is enabled: `system_profiler SPNVMeDataType | grep TRIM`

---

## 2. OpenCore Config Hardening (Production Mode)

- [ ] **Remove debug boot-args**: Change `keepsyms=1 debug=0x100` → remove or set `debug=0` in `NVRAM → Add → boot-args`
- [ ] **Disable verbose mode**: Remove `-v` from boot-args (optional, but cleaner)
- [ ] **Set `Misc → Debug → Target` to `3`** (or `0` to disable OC logging entirely) — reduces boot time
- [ ] **Set `Misc → Security → Vault` to `Optional`** if not already — prevents accidental lock-out
- [ ] **Verify `SecureBootModel`** — set to `Disabled` or `Default` as appropriate for AMD Ryzen
- [ ] **Confirm `ScanPolicy`** is set correctly to restrict boot entries to your drives
- [ ] **Backup your working EFI** to a USB drive or this repo before any changes

---

## 3. SMBIOS & Serial Numbers

- [ ] Verify your SMBIOS (`config.plist.tpl` uses `{{ROM}}` and `{{SystemSerialNumber}}` placeholders — confirm these are substituted in the deployed config)
- [ ] Run [**Apple Coverage**](https://checkcoverage.apple.com) or `system_profiler SPHardwareDataType` — serial should be unique and valid format but **not** registered to a real Mac
- [ ] Confirm `MLB` and `ROM` values are unique to avoid iCloud conflicts with other machines

---

## 4. iServices

- [ ] **iMessage / FaceTime**: Sign in and send a test message. If activation fails, regenerate MLB/ROM/Serial with **GenSMBIOS**
- [ ] **iCloud**: Sign in and verify **iCloud Drive**, **Photos**, and **Keychain** sync
- [ ] **App Store**: Sign in and install a free app to verify purchases work
- [ ] **AirDrop**: Test with an iPhone or another Mac (requires working WiFi as AirPort + BT)
- [ ] **Handoff / Continuity**: Test with an iPhone if BT is working

---

## 5. Sleep & Wake

- [ ] Test **sleep** (`Apple → Sleep`) and wake with keyboard/mouse — this is often the first thing to break on AMD hackintoshes
- [ ] Check sleep: `pmset -g` — confirm `hibernatemode` settings
- [ ] Disable features that break sleep if needed:
  ```bash
  sudo pmset -a hibernatemode 0    # disable hibernation (safe sleep)
  sudo pmset -a proximitywake 0    # disable Bluetooth wake
  sudo pmset -a tcpkeepalive 0     # disable TCP keepalive during sleep
  ```
- [ ] Test wake from sleep after 30 min idle (power-nap wake issues are common)
- [ ] Verify GPU/display wakes correctly (AMD-specific: may need `igfxonln=1` or similar WEG flags)

---

## 6. System Updates

- [ ] Before enabling **System Updates**: make sure your EFI is backed up
- [ ] Check that `SecureBootModel` and `ApECID` won't block OTA updates
- [ ] After any macOS update: **re-verify** all hardware still works before removing your fallback EFI USB
- [ ] Consider pinning to current Sonoma version until kexts (especially AirportBrcmFixup, AMDRyzenCPUPowerManagement) confirm Sonoma compatibility

---

## 7. Performance & Tuning

- [ ] Install **Stats** (free, open-source) — real-time GPU/CPU/RAM/NVMe sensors in menu bar
- [ ] Verify **AMD CPU power management** (P-states): install AMD Power Gadget and check frequency scaling under load vs idle
- [ ] Enable **HiDPI** if using a 1440p or 4K display: use `BetterDisplay` or `RDM`
- [ ] Set energy saver settings: **System Settings → Battery** (or Energy Saver on desktop) — tune as needed

---

## 8. Kext Audit (Cleanup)

Review the loaded kexts and disable/remove any that don't apply to your system:

- [ ] **AirportItlwm** vs **AirportBrcmFixup** — disable the one that doesn't match your WiFi card to avoid conflicts
- [ ] **NullCPUPowerManagement** — should be disabled if AMDRyzenCPUPowerManagement is working
- [ ] **AppleMCEReporterDisabler** — required for AMD dual-socket, keep enabled
- [ ] **CtlnaAHCIPort** — only needed for specific SATA controllers; verify if needed
- [ ] **SmallTreeIntel82576** — only needed if you have that specific Intel NIC; remove if not
- [ ] Verify all kexts in config.plist have `Enabled` = `true/false` correctly

---

## 9. Miscellaneous Quality-of-Life

- [ ] Set correct **Timezone** and enable automatic time sync (`systemsetup -setusingnetworktime on`)
- [ ] Install **Rosetta 2** if you need to run Intel-only apps: `softwareupdate --install-rosetta`
- [ ] Enable **FileVault** only after confirming everything is stable (requires NVRAM working correctly; can prevent boot if NVRAM is emulated)
- [ ] Set up **Time Machine** backup to an external drive — before exploring kext changes
- [ ] Install **OpenCore Configurator** or use **ProperTree** for future config edits
- [ ] Review and clean up **OpenCore boot picker** entries (`Misc → Entries`)

---

## 10. Known AMD Ryzen + Sonoma Issues to Watch

- **DRM**: Apple TV+, Netflix in Safari, and FairPlay DRM may not work (AMD dGPU only — no iGPU)
  - Workaround: use Chromium-based browsers for streaming
- **Virtualization (Docker/VMs)**: AMD requires `HypervisorFirmwareType` and Rosetta for some x86 Docker images
- **HandBrake VideoToolbox**: hardware H.264/H.265 encoding may not work — test and fall back to software
- **Universal Control / Sidecar**: Sidecar requires iGPU; won't work on AMD-only. Universal Control requires proper BT/WiFi

---

## Quick Diagnostic Commands

```bash
# Check all loaded kexts
kextstat | grep -v com.apple

# Check CPU info
sysctl -n machdep.cpu.brand_string
sysctl hw.physicalcpu hw.logicalcpu

# NVMe and TRIM
system_profiler SPNVMeDataType | grep -E 'Model|TRIM|Capacity'

# Check IORegistry for GPU
ioreg -l | grep -i "model" | grep -i "radeon\|amd"

# Boot args currently active
nvram 7C436110-AB2A-4BBB-A880-FE41995C9F82:boot-args

# Check USB port mapping
system_profiler SPUSBDataType

# Ethernet
ifconfig en0 | grep 'status\|inet '

# Sleep/wake log (last 10 events)
pmset -g log | grep -E 'Sleep|Wake' | tail -10
```

---

_Last updated: first boot — Sonoma on MSI X570 / AMD Ryzen_
