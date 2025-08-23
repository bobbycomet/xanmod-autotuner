# xanmod-autotuner

> # Note: #
> Not an official tool, community-made
A hardened, XanMod-aware auto-tuner for Linux kernels, designed to optimize system performance for desktop responsiveness and gaming. This script is fully compatible with XanMod kernels but will apply a balanced, performance-oriented profile to any Linux kernel. This tool is a component of the upcoming Griffin OS but is available here as an MIT-licensed open-source project.

# Features

Robust Kernel Detection: Accurately identifies XanMod, XanMod-RT, XanMod-LTS, and generic kernels to apply the most appropriate tunings.

Adaptive Tuning: Automatically calculates optimal values for ZRAM size, swappiness, EarlyOOM thresholds, and Preload memory limits based on your system's detected RAM.

Network Tuning: Tunes the network stack by enabling Fair Queueing (fq) and, if supported, the BBR congestion control algorithm to improve network responsiveness and throughput.

Safe Execution: Features safe file writes, rigorous input validation, and a fallback to sensible defaults to prevent system instability.

Persistent Configuration: Saves the detected, calculated, and applied settings to /etc/griffin/xanmod-autotune.conf, allowing for user overrides.

Systemd Integration: Designed to be run automatically at boot via a systemd service, ensuring your tunings are always active.

# How It Works

The script runs at boot with root privileges. It checks your system's kernel and available RAM to determine a recommended set of tuning parameters. It then applies these settings using standard Linux utilities like sysctl, zram, and earlyoom. The script ensures that ZRAM is enabled, swappiness is configured for optimal performance, and that services like earlyoom and preload are configured with ideal thresholds.

# Installation and Usage

The recommended way to install xanmod-autotuner is by adding our apt repository and installing it with apt.

Add the APT Repository and GPG Key

First, import the GPG key used to sign the packages. This ensures that the packages you download are authentic.


```
sudo mkdir -p /etc/apt/keyrings
wget -O- https://bobbycomet.github.io/xanmod-autotuner/dists/xanmod/Release.gpg | gpg --dearmor | sudo tee /etc/apt/keyrings/xanmod-autotuner.gpg >/dev/null
```
Next, add the repository to your system's sources list.


```
echo "deb [signed-by=/etc/apt/keyrings/xanmod-autotuner.gpg] https://bobbycomet.github.io/xanmod-autotuner/ xanmod main" | sudo tee /etc/apt/sources.list.d/xanmod-autotuner.list
```

Finally, update your package list to include the new repository.


```
sudo apt update
```

# Install the Package

Now, you can install the xanmod-autotuner package using apt.

```
sudo apt install xanmod-autotuner
```

The package will automatically install the script, set up the systemd service to run it at boot, and apply the tunings immediately.

# Verify the Installation

To confirm that the script ran successfully, you can check the status of the systemd service.


```
sudo systemctl status xanmod-autotuner.service
```

You can also inspect the generated configuration file to see the values that were applied.

```
cat /etc/griffin/xanmod-autotune.conf
```

# Advanced: Custom Overrides

If you want to manually change a tuning value, you can edit the configuration file and set an override. For example, to set a fixed swappiness value of 10, you would edit /etc/griffin/xanmod-autotune.conf and change the line OVERRIDE_SWAPPINESS="" to OVERRIDE_SWAPPINESS="10".

After saving the file, simply run the script manually to apply the changes.

```
sudo /usr/local/bin/autotuner.sh
```
> #Note#: If you want to use the script from a different location, ensure you use the correct path.

# Example: 

:~$ cat /etc/griffin/xanmod-autotune.conf
# Griffin OS XanMod Auto-Tune Config
# - Detected values (do not edit)
DETECTED_KERNEL_RELEASE="6.11.0-26-generic"

DETECTED_KERNEL_BASE="6.11.0"

DETECTED_KERNEL_FLAVOR="generic"

DETECTED_KERNEL_VARIANT=""

DETECTED_KERNEL_PKG="non-xanmod"

DETECTED_RAM_MB="32039"

DETECTED_RAM_GB="32"

# - Calculated defaults (based on kernel+RAM)
CALC_ZRAM_PCT="33"

CALC_ZRAM_COMP="lz4"

CALC_SWAPPINESS="60"

CALC_EARLYOOM_MEM="5"

CALC_EARLYOOM_SWAP="5"

CALC_PRELOAD_MEMLIMIT="70"

CALC_PRELOAD_ENABLE="1"

# - Overrides (user-editable; leave empty to use calculated)

OVERRIDE_ZRAM_PERCENT=""

OVERRIDE_ZRAM_COMP=""

OVERRIDE_SWAPPINESS=""

OVERRIDE_EARLYOOM_MEM=""

OVERRIDE_EARLYOOM_SWAP=""

OVERRIDE_PRELOAD_MEMLIMIT=""

OVERRIDE_PRELOAD_ENABLE=""

# - Effective (applied this run)
EFFECTIVE_ZRAM_PCT="33"

EFFECTIVE_ZRAM_COMP="lz4"

EFFECTIVE_SWAPPINESS="60"

EFFECTIVE_EARLYOOM_MEM="5"

EFFECTIVE_EARLYOOM_SWAP="5"

EFFECTIVE_PRELOAD_MEMLIMIT="70"

EFFECTIVE_PRELOAD_ENABLE="1"

:~$ swapon --show

NAME       TYPE      SIZE USED PRIO

/swapfile  file        2G   0B   -2

/dev/zram0 partition  64M 7.5M  100

:~$ cat /proc/sys/vm/swappiness

60


:~$ cat /etc/sysctl.d/99-griffin-tune.conf

# /etc/sysctl.d/99-griffin-tune.conf - auto-generated by xanmod-autotune.sh

# Effective tuning for kernel=6.11.0-26-generic flavor=generic variant=none

vm.swappiness=60

vm.vfs_cache_pressure=100

net.core.default_qdisc=fq


