---
layout: post
title: Bypassing Windows 11 TPM 2.0 requirement (Pt 2/2)
---

The leaked build of the upcoming Windows 11 (build 21996.1) has a TPM 2.0 requirement. We do not know if
this is going to be the case when it actually launches in the fall, but for now, I was trying to get around
the limitation. I was trying to install it on my 7 year old laptop, which unfortunately does not have a
Trusted Platform Module (TPM) chip in it.

If you are going to follow these steps, I bear no responsibility for any security compromises or damages 
to your machine. This is for informational and experimental purposes only.

Couple of things to keep in mind:
1. Do not do this on your primary system.
2. Do not use your primary Microsoft Account to log in to this Windows.
3. If the TPM 2.0 requirement is actually there for production builds (I doubt it will be), do not follow this to bypass that requirement, since that may compromise security of the operating system.

## Installation attempt 3

### Step 1: Preparing the space
Open **Disk Management**. Right click on the partition with the Windows 10 install and choose _Shrink Volume_.
Enter a minimum size of 50GB to shrink. In my case, I had plenty of free space, so I shrunk 250GB.

Right click on the unallocated space and choose _Create Simple Partition_. Select the available size, format
the partition as NTFS, and assign a drive letter `D:`.

Now the partition layout looks similar to this:

| `EFI System Partition` | `C: (Windows 10)` | `D: (Empty)` | `Recovery Partition` |
|:----------------------:|:-----------------:|:------------:|:--------------------:|
|                        |                   |              |                      |

### Step 2: Laying the image
Double-click the Windows 11 ISO to mount it. In my case it loaded as `E:` drive.
Open **Powershell** as Administrator. The prompt should say: `PS C:\Windows\System32>`.

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

~~~
DISKPART> list disk

  Disk ###  Status         Size     Free     Dyn  Gpt
  --------  -------------  -------  -------  ---  ---
  Disk 0    Online          465 GB  3072 KB        *

DISKPART> select disk 0

Disk 0 is now the selected disk.

DISKPART> list partition

  Partition ###  Type              Size     Offset
  -------------  ----------------  -------  -------
  Partition 1    System             100 MB  1024 KB
  Partition 2    Reserved            16 MB   101 MB
  Partition 3    Primary            215 GB   117 MB
  Partition 4    Primary            249 GB   215 GB
  Partition 5    Recovery           499 MB   465 GB

DISKPART> select partition 1

Partition 1 is now the selected partition.

DISKPART> assign letter="S"

DiskPart successfully assigned the drive letter or mount point.

DISKPART>
~~~

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
