---
layout: post
title: Bypassing Windows 11 TPM 2.0 requirement (Pt 2/2)
---

## Installation attempt 3

### Step 1: Preparing the space
Open Disk Management. Right click on the partition with the Windows 10 install and choose Shrink Volume.
Enter a minimum size of 50GB to shrink. In my case, I had plenty of free space, so I shrunk 250GB.

Right click on the unallocated space and choose Create Simple Partition. Select the available size, format
the partition as NTFS, and assign a drive letter `D:`.

Now the partition layout looks similar to this:

| `EFI System Partition` | `C: (Windows 10)` | `D: (Empty)` | `Recovery Partition` |
|:----------------------:|:-----------------:|:------------:|:--------------------:|
|                        |                   |              |                      |

### Step 2: Laying the image
Double-click the Windows 11 ISO to mount it. In my case it loaded as `E:` drive.
Open Powershell as Administrator. The prompt should say: `PS C:\Windows\System32>`.

Navigate to the `sources` folder in the ISO in powershell.

```
PS C:\Windows\System32> E:
PS E:\> cd sources
PS E:\sources>
```

Now use the Deployment Image Servicing and Management (`dism`) tool to lay the Windows 11 image on to the newly created `D:` drive.

```
PS E:\sources> Dism /Apply-Image /ImageFile:install.wim /index:1 /ApplyDir:D:\

Deployment Image Servicing and Management tool
Version 10.0.xxxxx.xxx
... 
```

Wait till the operation is successful.

### Step 3: Create a boot entry
Load `diskpart` in powershell, and assign a partition letter to the EFI system partition.

```
PS E:\sources> C:
PS C:\Windows\System32> diskpart
```

Select the partition that says `System` and assign letter `S` to it.

```
DISKPART> list disk
DISKPART> select disk 0
DISKPART> list partition
DISKPART> select partition 1
DISKPART> assign letter="S"
DISKPART> exit
PS C:\Windows\System32>
```

Now navigate to the Windows 11 install folder and use `bcdboot` in powershell to copy the boot files to the EFI bootloader.

```
PS C:\Windows\System32> D:
PS D:\Windows\System32> bcdboot D:\Windows /s S: /f UEFI
Boot files copied successfully.
PS D:\Windows\System32> exit
```

Almost there!

### Step 4: Reboot
Restart the machine. Now, at boot time, there will be two `Windows 10` entries. The top one with a higher volume
number will most likely be the Windows 11 install. Select that and boot it up.

Windows 11 will set up devices and services and launch the new OOBE experience. Go through the setup process. 
I highly recommend not using the primary Microsoft account to log in (since this is not a build coming from a trusted source). 
Create a new Microsoft account or else do not connect to WiFi and choose a local account instead. Once the OOBE is complete, the new Windows 11 experience will launch.

Have fun!