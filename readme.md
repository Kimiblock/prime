A utility aimed to achieve dual graphics switching with very low performance impact and automatically turns off NVIDIA GPUs to save power.

Install `prime` and required packages, run `prime --first-use` in your terminal, then enjoy it~

# Install

## Required packages (For folks who are using arch)

```pacman
lib32-nvidia-utils #For 32-bit applications
nvidia #(nvidia-open, though not tested)
nvidia-utils #For nvidia-smi
cronie
polkit #For permission elevation, use with agents (example: polkit-kde-agent)
nvidia-settings #(Optional)
screen
```

## Install `prime`

Open your terminal:

```bash
curl -O https://raw.githubusercontent.com/Kimiblock/prime/master/prime
sudo chown $USER:root prime
sudo mv prime /sbin
sudo chmod 755 /sbin/prime
prime --first-use
```

# Start Applications on dGPU

## Most Applications

```bash
_render=[all, display, vulkan] prime [command]
```

| Syntax (_render) | Effects                                                      |
| ---------------- | ------------------------------------------------------------ |
| all              | Run Applications completely on `dGPU`                        |
| display          | Render you application on `dGPU` (Similar to prime-run)      |
| vulkan           | Run only `vulkan` on `dGPU`  ( Works on `dxvk` games like `GTAV` on Proton ) |

## Steam games

Edit you properties, add

```bash
_render=vulkan prime %command% #for dxvk games
```

```bash
_render=all prime %command% #for other games
```

If you are encountering weird stutters with AMD APU + Nvidia, replace the above to

```bash
DXVA_ASYNC=1 PROTON_NO_ESYNC=1 RADV_DEBUG=llvm ENABLE_VKBASALT=0 _render=vulkan prime %command%
```

may help.

# How does `prime` work?

## Boot process

When your computer boots, `nvidia` module are blacklisted so that they won't be loaded.

Then, the `crontab` rule added by `prime` will enable kernel power management for your dGPU.

## Offload applications

Once you call the command `prime`, it loads the `nvidia` kernel modules to offload Applications on your dGPU.

## Deactivate `dGPU`

After all applications running through `prime` shutdown, it removes the `nvidia` kernel module so that other applications won't activate your graphics card from time to time. (As far as I know, Firefox activates `dGPU` every time when a video is being played.)

If any applications are using `dGPU` causing `modprobe -r` to fail, `prime` will send a desktop notification and elevate its permission to kill that process.

## Advanced

<mark> If you do want to manually modprobe the kernel module, add `--ignore-install` </mark>
