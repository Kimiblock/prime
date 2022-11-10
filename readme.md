A bash script that can power down `Nvidia Optimus` cards automatically, inspired by [this post](https://www.reddit.com/r/linux_gaming/comments/sqqzl4/nvidia_primerun_on_wayland_instead_of_nvidiaxrun/)

# Install

## Required packages

```pacman
lib32-nvidia-utils
nvidia
nvidia-utils
nvidia-xrun-pm
nvidia-prime (Optional)
nvidia-settings (Optional)
```

## Install `prime`

Download the executable 'prime' and copy it into `/sbin`

## Configuration

### ~~Optimus Manager~~

~~Edit `/etc/optimus-manager/optimus-manager.conf`~~

### nvidia-xrun

#### Service

```bash
sudo systemctl enable nvidia-xrun-pm.service
```



#### Conf file

Add `/etc/modprobe.d/blacklist-nvidia.conf` to blacklist Nvidia kernel modules

```blacklist-nvidia.conf
blacklist nvidia
blacklist nvidia-drm
blacklist nvidia-modeset
blacklist nvidia_uvm
install nvidia /bin/false
```

<mark> If you want to modprobe the kernel module, please add `--ignore-install` </mark>

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

# Advanced

## Change the BUS ID

Follow [this guide](https://github.com/Witko/nvidia-xrun#setting-the-right-bus-id) to change your BUS ID for `nvidia-xrun`

Edit the prime executable, replace all the `03:00.0` to your BUS ID, which can be found by 

```bash
lspci | grep -i nvidia | awk '{print $1}'
```



This will enable `prime` to apply proper power control to your card

# How does `prime` work?

`Prime` relies on `optimus-manager` to turn off dGPU when your computer boots.

Once you call the command `prime`, it tells the Linux kernel to rescan all PCIe devices and load Nvidia kernel modules to pass some essential syntax to offload on your dGPU.

After all applications running through `prime` shutdown, it will remove the Nvidia kernel modules so that other  applications won't use your graphics card from time to time. (As I know, Firefox activates my GPU when a video is being played and Chromium activates my GPU every time when it starts)

