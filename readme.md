A bash script that can power down `Nvidia Optimus` cards automatically, inspired by [this post](https://www.reddit.com/r/linux_gaming/comments/sqqzl4/nvidia_primerun_on_wayland_instead_of_nvidiaxrun/)

# Install

## Required packages

```pacman
lib32-nvidia-utils
nvidia (nvidia-open, though not tested)
nvidia-utils
cronie
nvidia-settings (Optional)
```

## Install `prime`

Download the executable 'prime' and copy it into `/sbin`

## Configuration

### Blacklist Module

Add `/etc/modprobe.d/blacklist-nvidia.conf` to blacklist Nvidia kernel modules

```blacklist-nvidia.conf
blacklist nvidia
blacklist nvidia-drm
blacklist nvidia-modeset
blacklist nvidia_uvm
blacklist nouveau
install nvidia /bin/false
```

<mark> If you do want to manually modprobe the kernel module, add `--ignore-install` </mark>

### Add a `cronie` rule

#### Get your Bus id

```bash
lspci | grep -i nvidia | awk '{print $1}'
```

#### Edit `crontab`

```bash
sudo crontab -e
```

Add this line:

```bash
@reboot bash -c "sleep 5s && echo 'auto' > '/sys/bus/pci/devices/0000:03:00.0/power/control'"
```

Where `03:00.0` is your Bus id.

# Start Applications on dGPU

## Most Applications

```bash
_render=[all, display, vulkan] prime [command]
```

| Syntax (_render) | Effects                                                      |
| ---------------- | ------------------------------------------------------------ |
| all              | Run Applications completely on dGPU                          |
| display          | Render you application on dGPU (Similar to prime-run)        |
| vulkan           | Run `vulkan` on dGPU  ( Works on `dxvk` games like `GTAV` on Proton ) |

## Steam games

Edit you properties, add

```bash
_render=vulkan prime %command%
```

If you are encountering weird stutters, replace to

```bash
DXVA_ASYNC=1 PROTON_NO_ESYNC=1 RADV_DEBUG=llvm ENABLE_VKBASALT=0 _render=vulkan prime %command%
```

may help.

# Advanced

## Correct Bus ID

Follow [this guide](https://github.com/Witko/nvidia-xrun#setting-the-right-bus-id) to change your BUS ID for `nvidia-xrun`

Edit the prime executable, replace all the `03:00.0` to your BUS ID, which can be found by 

```bash
lspci | grep -i nvidia | awk '{print $1}'
```



This will enable `prime` to apply proper power control to your card

# How does `prime` work?

## Boot process

When your computer boots, `nvidia` module are blacklisted so that they won't be loaded.

Then, the `crontab` rule added before will enable kernel's default power management for your dGPU.

## Offload applications

Once you call the command `prime`, it loads the `nvidia` kernel modules to offload Applications on your dGPU.

## Deactivate `dGPU`

After all applications running through `prime` shutdown, it will remove the `nvidia` kernel module so that other applications won't activate your graphics card from time to time. (As far as I know, Firefox activates `dGPU` every time when a video is being played.)

If any applications are using `dGPU` causing `modprobe -r` to fail, `prime` will send a desktop notification and elevate its permission to kill that process.
