A bash script that can power down `Nvidia Optimus` cards automatically, inspired by [this post](https://www.reddit.com/r/linux_gaming/comments/sqqzl4/nvidia_primerun_on_wayland_instead_of_nvidiaxrun/)

# Install

## Required packages

```pacman
lib32-nvidia-utils
nvidia
nvidia-utils
bbswitch (bbswitch-dkms)
acpi_call
optimus-manager (optimus-manager-qt, for GUI)
nvidia-prime (Optional)
nvidia-settings (Optional)
```

## Install `prime`

Download the executable 'prime' and copy it into `/sbin`

## Configuration

### Optimus Manager

Edit `/etc/optimus-manager/optimus-manager.conf`

```optimus-manager.conf
[nvidia]
DPI=0
PAT=yes
allow_external_gpus=yes
dynamic_power_management=fine
ignore_abi=no
modeset=yes
options=

[optimus]
auto_logout=no
pci_power_control=no
pci_remove=yes
pci_reset=function_level
startup_auto_battery_mode=integrated
startup_auto_extpower_mode=integrated
startup_mode=integrated
switching=bbswitch
```

### Environment (APUs)

`/etc/environment`

Add

```environment
VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json
```



# Start Applications on dGPU

```bash
_render=[all, display, vulkan] prime [command]
```

| Syntax (_render) | Effects                                                      |
| ---------------- | ------------------------------------------------------------ |
| all              | Render your application and process Vulkan commands on your dGPU |
| display          | Render you application on dGPU (Similar to prime-run)        |
| vulkan           | Process vulkan commands on dGPU ( Works on `dxvk` games like `GTAV` on Proton ) |

# How does `prime` work?

`Prime` relies on `optimus-manager` to turn off dGPU when your computer boots.

Once you call the command `prime`, it tells the Linux kernel to rescan all PCIe devices and load Nvidia kernel modules to pass some essential syntax to offload on your dGPU.

After all applications running through `prime` shutdown, it will remove the Nvidia kernel modules so that other  applications won't use your graphics card from time to time. (As I know, Firefox activates my GPU when a video is being played and Chromium activates my GPU every time when it starts)

