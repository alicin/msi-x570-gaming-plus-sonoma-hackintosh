![macOS Sonoma 14.1.1](https://raw.githubusercontent.com/alicin/msi-x570-gaming-plus-sonoma-hackintosh/main/ss.png)

### Specs

- MoBo: MSI MPG X570 Gaming Plus
- CPU: AMD Ryzen 5700G 16Core CPU
- RAM: 32GB
- Graphics Card: Sapphire 6800XT
- WiFi/BT: Fenvi BCM BCM94360CD (Run Post-Install Root Patch from OpenCore Legacy Patcher after installation to enable)
- OpenCore: 0.9.6

### BIOS Setting

- Secure Boot: Disable
- Serial Port: Disable
- XHCI Hand-off: Enable
- Legacy USB Support: Enable

### NOTES

- Handoff, iMessage/Facetime works
- Sleep works
- Virtualization doesn't work because amd
- Sidecar doesn't work either (Maybe with USB mapping?)
- iGPU could be achieved with [NootedRed](https://github.com/ChefKissInc/NootedRed)
- Post-Install Root Patch from OpenCore Legacy Patcher should be run after installation to enable WiFi
