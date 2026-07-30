---
title: Filesystem Mounting Cheat Sheet
date: 2026-07-30
image:
  path: assets/img/blog/filesystem_evidence.png
categories: [Blog]
tags: [blog, linux, dfir, digital-forensics, disk-imaging, filesystem-mounting, lvm, ewf-e01, disk-encryption, proxmox, cheat-sheet]
---

> A consolidated reference for mounting disk images, virtual disks and forensic
> acquisitions on Linux: raw images, LVM volumes, EWF/E01 evidence files, VMDK
> virtual disks, Proxmox `.vma` backups, plus encrypted volumes (LUKS and
> BitLocker) and the correct teardown procedure for each.
>
> All file, disk, volume group and mountpoint names in this document are
> placeholders (`disk1.raw`, `image1.E01`, `vg1`, `lv1`, …). Replace them with
> the real names returned by your own commands.

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [1. Core Concepts](#1-core-concepts)
  - [1.1 The general workflow](#11-the-general-workflow)
  - [1.2 Sectors and byte offsets](#12-sectors-and-byte-offsets)
  - [1.3 The LVM hierarchy](#13-the-lvm-hierarchy)
  - [1.4 Read-only golden rules](#14-read-only-golden-rules)
- [2. Required Tools](#2-required-tools)
- [3. Inspecting a Disk Image](#3-inspecting-a-disk-image)
- [4. Mounting Raw Images](#4-mounting-raw-images)
  - [4.1 Method A — mount with an offset](#41-method-a--mount-with-an-offset)
  - [4.2 Method B — loop device with partition scanning](#42-method-b--loop-device-with-partition-scanning)
  - [4.3 Method C — kpartx partition mapping](#43-method-c--kpartx-partition-mapping)
- [5. LVM Volumes](#5-lvm-volumes)
  - [5.1 LVM command reference](#51-lvm-command-reference)
  - [5.2 Activating and mounting a logical volume](#52-activating-and-mounting-a-logical-volume)
  - [5.3 Partial activation (missing physical volume)](#53-partial-activation-missing-physical-volume)
- [6. EWF / E01 Forensic Images](#6-ewf--e01-forensic-images)
  - [6.1 Mounting the EWF container](#61-mounting-the-ewf-container)
  - [6.2 Mapping the partitions](#62-mapping-the-partitions)
  - [6.3 Mounting a plain partition](#63-mounting-a-plain-partition)
  - [6.4 Full walkthrough: LVM inside an EWF image](#64-full-walkthrough-lvm-inside-an-ewf-image)
    - [Step 1 — Mount the EWF image](#step-1--mount-the-ewf-image)
    - [Step 2 — Create the loop devices](#step-2--create-the-loop-devices)
    - [Step 3 — Scan and activate the volume group](#step-3--scan-and-activate-the-volume-group)
    - [Step 4 — Identify the filesystem](#step-4--identify-the-filesystem)
    - [Step 5a — ext4 with a dirty journal](#step-5a--ext4-with-a-dirty-journal)
    - [Step 5b — XFS with a dirty journal](#step-5b--xfs-with-a-dirty-journal)
    - [Step 5c — LUKS-encrypted logical volume](#step-5c--luks-encrypted-logical-volume)
    - [Step 6 — Analysis](#step-6--analysis)
    - [Step 7 — Unmount](#step-7--unmount)
- [7. Encrypted Volumes](#7-encrypted-volumes)
  - [7.1 LUKS](#71-luks)
  - [7.2 BitLocker with dislocker](#72-bitlocker-with-dislocker)
- [8. VMDK Virtual Disks](#8-vmdk-virtual-disks)
  - [8.1 Method 1 — qemu-nbd (recommended)](#81-method-1--qemu-nbd-recommended)
  - [8.2 Method 2 — vmware-mount](#82-method-2--vmware-mount)
  - [8.3 Method 3 — guestmount (libguestfs)](#83-method-3--guestmount-libguestfs)
- [9. Proxmox `.vma` Backups](#9-proxmox-vma-backups)
  - [9.1 Getting the vma tool](#91-getting-the-vma-tool)
  - [9.2 Listing the archive contents](#92-listing-the-archive-contents)
  - [9.3 Extracting the disk images](#93-extracting-the-disk-images)
- [10. Unmounting and Cleanup](#10-unmounting-and-cleanup)
  - [10.1 Teardown order](#101-teardown-order)
  - [10.2 Step-by-step procedure](#102-step-by-step-procedure)
  - [10.3 Final verification](#103-final-verification)
- [11. Troubleshooting](#11-troubleshooting)
- [12. Mount Options Quick Reference](#12-mount-options-quick-reference)
- [13. Flow Diagrams](#13-flow-diagrams)

---

## 1. Core Concepts

### 1.1 The general workflow

Almost every scenario in this document follows the same four stages:

1. **Expose the image as a block device** — `losetup`, `kpartx`, `ewfmount`,
   `qemu-nbd`, or a `mount -o loop,offset=…`.
2. **Discover what is inside it** — `fdisk`, `mmls`, `lsblk`, `blkid`,
   `vgscan`.
3. **Unlock any layer that sits between you and the filesystem** — activate the
   LVM volume group, open the LUKS container, decrypt the BitLocker volume.
4. **Mount the filesystem read-only**, work on it, then **tear everything down
   in reverse order**.

### 1.2 Sectors and byte offsets

Partition tables report the start of a partition in **sectors**, while `mount`
expects a **byte** offset. Multiply by the sector size (512 bytes on virtually
all images you will meet):

```
byte_offset = start_sector × 512
```

Example — mounting the third partition of `disk1.raw`:

```
Device        Start      End    Sectors  Size  Type
disk1.raw1     2048   395263     393216  192M  EFI System
disk1.raw2   395264  1050623     655360  320M  Linux filesystem
disk1.raw3  1050624 11012095    9961472  4.8G  Linux filesystem
disk1.raw4 11012096 20969471    9957376  4.7G  Linux swap
```

`disk1.raw3` is the operating system partition, so:

```
1050624 × 512 = 537919488
```

> If `fdisk -l` reports a sector size other than 512 bytes (it prints
> `Sector size (logical/physical)`), use that value instead.

### 1.3 The LVM hierarchy

- **PV — Physical Volume**: a disk or partition handed over to LVM
  (for example `/dev/sda2`, `/dev/loop0p3`).
- **VG — Volume Group**: a pool made of one or more PVs (for example `vg1`).
- **LV — Logical Volume**: a volume carved out of a VG — this is what you
  actually mount (for example `/dev/vg1/lv1`).

```
PV (Physical Volume) → VG (Volume Group) → LV (Logical Volume)
```

You can never mount a PV or a VG directly. Trying to mount the raw LVM
partition gives `unknown filesystem type 'LVM2_member'`; trying to mount the VG
gives `/dev/vg1 is not a block device`.

### 1.4 Read-only golden rules

When working on evidence or on any image you cannot afford to alter:

- Always mount with `-o ro`.
- Attach loop devices read-only with `losetup -r`.
- Never let the kernel replay a dirty journal: add `noload` for ext3/ext4 or
  `norecovery` for XFS. Without it the kernel tries a write-mode recovery and
  the mount fails on a read-only device.
- Do not `chroot` into a forensic acquisition on a production system.
- Prefer a hardware or software write blocker for the source media.

---

## 2. Required Tools

```bash
# Debian / Ubuntu
sudo apt install ewf-tools lvm2 util-linux kpartx qemu-utils \
                 libguestfs-tools cryptsetup dislocker

# Fedora / RHEL
sudo dnf install libewf-tools lvm2 util-linux kpartx qemu-img \
                 libguestfs-tools cryptsetup
```

| Tool                      | Purpose                                                  |
| ------------------------- | -------------------------------------------------------- |
| `ewf-tools` (`ewfmount`)  | Exposes `.E01`/EWF evidence files as a raw image         |
| `lvm2`                    | `pvscan`, `vgscan`, `lvs`, `vgchange`, `lvdisplay`       |
| `util-linux`              | `losetup`, `mount`, `lsblk`, `blkid`, `fdisk`            |
| `kpartx`                  | Maps partitions of an image to `/dev/mapper/` devices    |
| `qemu-utils` / `qemu-img` | `qemu-nbd` for VMDK, QCOW2, VDI, VHD                     |
| `libguestfs-tools`        | `guestmount` / `guestunmount`, filesystem-aware mounting |
| `cryptsetup`              | LUKS containers                                          |
| `dislocker`               | BitLocker-encrypted volumes                              |
| `sleuthkit`               | `mmls` for partition layouts of forensic images          |

---

## 3. Inspecting a Disk Image

Before mounting anything, find out what the image contains.

```bash
# Partition table
fdisk -l disk1.raw

# Partition layout, forensic-oriented (Sleuth Kit)
mmls disk1.raw

# Block device tree, once a loop/nbd device exists
lsblk /dev/loop0

# Filesystem type, label and UUID — run this before every mount
blkid /dev/mapper/vg1-lv1
```

`blkid` is the deciding step: the `TYPE=` it returns tells you which mount
command to use.

| `TYPE=` returned | Situation                                        | Mount command                                           |
| ---------------- | ------------------------------------------------ | ------------------------------------------------------- |
| `ext4`           | Dirty journal (typical of forensic acquisitions) | see [§6.4 step 5a](#step-5a--ext4-with-a-dirty-journal) |
| `xfs`            | Dirty journal                                    | see [§6.4 step 5b](#step-5b--xfs-with-a-dirty-journal)  |
| `crypto_LUKS`    | Encrypted volume                                 | see [§7.1](#71-luks)                                    |
| `LVM2_member`    | LVM physical volume                              | see [§5](#5-lvm-volumes)                                |
| `BitLocker`      | BitLocker volume                                 | see [§7.2](#72-bitlocker-with-dislocker)                |

---

## 4. Mounting Raw Images

### 4.1 Method A — mount with an offset

The quickest route when you only need **one** partition and there is no LVM in
the way.

```bash
# 1. Read the partition table and pick the partition
fdisk -l disk1.raw

# 2. Compute the byte offset: start_sector × 512
#    e.g. 1050624 × 512 = 537919488

# 3. Create the mountpoint
sudo mkdir -p /mnt/vmroot0

# 4. Mount
sudo mount -o loop,offset=537919488 ./disk1.raw /mnt/vmroot0

# Read-only variant (recommended on evidence)
sudo mount -o loop,ro,noload,offset=537919488 ./disk1.raw /mnt/vmroot0
```

Unmount:

```bash
sudo umount /mnt/vmroot0
```

### 4.2 Method B — loop device with partition scanning

Better when the image holds several partitions or any LVM. `-P` makes the
kernel scan the partition table and create `/dev/loop0p1`, `/dev/loop0p2`, …

```bash
# Attach and let the kernel pick a free loop device; --show prints its name
sudo losetup -Pf --show disk1.raw
# → /dev/loop0

# Read-only attach (forensics): -r
sudo losetup -Prf disk1.raw

# Inspect the partitions that appeared
lsblk /dev/loop0
sudo fdisk -l /dev/loop0

# Mount one of them
sudo mkdir -p /mnt/part1
sudo mount -o ro,noload /dev/loop0p3 /mnt/part1
```

| Flag     | Meaning                                                  |
| -------- | -------------------------------------------------------- |
| `-f`     | Use the first free loop device                           |
| `--show` | Print the device name that was assigned                  |
| `-P`     | Scan the partition table and create the `pN` sub-devices |
| `-r`     | Attach read-only                                         |
| `-l`     | List the currently attached loop devices                 |
| `-d`     | Detach one loop device                                   |
| `-D`     | Detach **all** loop devices                              |

### 4.3 Method C — kpartx partition mapping

An alternative to `losetup -P`; it creates the partition devices under
`/dev/mapper/` instead.

```bash
sudo kpartx -av disk1.raw
# → add map loop0p1 …
#   add map loop0p2 …
#   add map loop0p3 …

sudo mkdir -p /mnt/part1
sudo mount -o ro,noload /dev/mapper/loop0p3 /mnt/part1
```

The operating system usually lives on the **largest** partition — a good first
guess when the layout is unfamiliar.

Remove the mappings when done:

```bash
sudo kpartx -dv disk1.raw
```

---

## 5. LVM Volumes

### 5.1 LVM command reference

| Command             | What it does                                        |
| ------------------- | --------------------------------------------------- |
| `pvscan`            | Finds the **Physical Volumes** available            |
| `vgscan`            | Detects the **Volume Groups** present on those PVs  |
| `vgs`               | Lists the Volume Groups in a compact table          |
| `lvs`               | Lists the **Logical Volumes** in a compact table    |
| `lvdisplay`         | Detailed LV output (UUID, path, status, attributes) |
| `vgchange -ay <vg>` | Activates every LV in the group                     |
| `vgchange -an <vg>` | Deactivates the group                               |

```bash
sudo pvscan
```

```
PV /dev/loop0p3   VG vg1   lvm2 [20.00 GiB / 0 free]
```

```bash
sudo vgscan
```

```
Found volume group "vg1" using metadata type lvm2
```

```bash
sudo lvs
```

```
LV    VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
lv1   vg1  -wi-ao----  10.00g
lv2   vg1  -wi-ao----   5.00g
```

```bash
sudo lvdisplay
```

```
--- Logical volume ---
LV Path                /dev/vg1/lv1
LV Name                lv1
VG Name                vg1
LV UUID                XxYy-Zz123-...
LV Write Access        read/write
LV Status              available
```

Reading the `Attr` field of `lvs`: the 5th character is `a` when the volume is
**active** and `-` when it is not; a trailing `p` means the volume is
**partial**, i.e. one of its physical volumes is missing.

### 5.2 Activating and mounting a logical volume

```bash
# 1. Expose the image and its partitions
sudo losetup -Pf --show disk2.raw
# → /dev/loop0
lsblk /dev/loop0

# 2. Find the volume group
sudo vgscan
# → Found volume group "vg1" using metadata type lvm2

# 3. Inspect the logical volumes
sudo lvs

# 4. Activate the volume group
sudo vgchange -ay vg1

# 5. Create the mountpoint and mount the LV (not the VG!)
sudo mkdir -p /mnt/vmroot
sudo mount /dev/vg1/lv1 /mnt/vmroot/

# Read-only variant
sudo mount -o ro,noload /dev/mapper/vg1-lv1 /mnt/vmroot/
```

> The VG name is case-sensitive and includes its underscores. Use exactly the
> string returned by `vgscan` — `vg_data1`, not `vgdata1`.

Both `/dev/vg1/lv1` and `/dev/mapper/vg1-lv1` refer to the same device.

### 5.3 Partial activation (missing physical volume)

When the volume group spans several disks and you only have some of them —
common with images extracted one disk at a time — LVM refuses to activate the
group and warns:

```
WARNING: Couldn't find device with uuid AaBbCc-1234-DdEe-5678-FfGg-9012-HhIiJj.
WARNING: VG vg1 is missing PV AaBbCc-1234-DdEe-5678-FfGg-9012-HhIiJj (last written to /dev/sdb5).
Found volume group "vg1" using metadata type lvm2
```

Force activation with the data that is available:

```bash
sudo vgchange -ay vg1 --activationmode partial
```

`lvs` then shows the volume with a `p` (partial) attribute:

```
LV    VG   Attr       LSize
lv1   vg1  -wi-----p- 179.98g
```

The volume mounts, but any extent living on the missing PV reads back as
zeroes. Expect truncated or unreadable files, and note the limitation in your
report.

---

## 6. EWF / E01 Forensic Images

EWF (`.E01`, `.Ex01`) is the Expert Witness Format produced by EnCase, FTK
Imager and `ewfacquire`. `ewfmount` presents the whole segmented set as a
single raw file called `ewf1`.

### 6.1 Mounting the EWF container

```bash
sudo apt install ewf-tools

sudo mkdir -p /mnt/ewf

# Point at the first segment; ewfmount picks up E02, E03, … automatically
sudo ewfmount image1.E01 /mnt/ewf/
```

**Check:**

```bash
ls /mnt/ewf/            # ewf1 must appear
fdisk -l /mnt/ewf/ewf1  # verify the partition table
```

`ewfmount` is a FUSE mount and is always read-only — the raw image it exposes
cannot be written to.

### 6.2 Mapping the partitions

Either of the two approaches works:

```bash
# Option A — loop device, read-only, with partition scanning
sudo losetup -Prf /mnt/ewf/ewf1

# Which loop device was assigned?
losetup -l | grep ewf1
lsblk | grep loop
```

```bash
# Option B — kpartx
sudo kpartx -av /mnt/ewf/ewf1
```

You now have `/dev/loopXX` with `/dev/loopXXp1`, `/dev/loopXXp2`, … (option A)
or `/dev/mapper/loopXXp1`, … (option B).

**Check:**

```bash
sudo fdisk -l /dev/loopXX   # must show the same partitions as the EWF image
lsblk /dev/loopXX           # is LVM already detected?
```

### 6.3 Mounting a plain partition

If the partition holds an ordinary filesystem rather than an LVM PV:

```bash
sudo mkdir -p /mnt/part
sudo mount /dev/mapper/loopXXp3 /mnt/part
```

If that fails with a read-only or superblock error, force the safe options:

```bash
sudo mount -o ro,norecovery /dev/loopXX /mnt/part/
```

`norecovery` (and `noload` on ext3/ext4) is essential for filesystems with an
uncommitted journal — the usual state of a forensic acquisition. Without it the
kernel attempts a write-mode journal recovery and fails because the device is
read-only.

### 6.4 Full walkthrough: LVM inside an EWF image

The complete read-only procedure, from evidence file to mounted filesystem.

#### Step 1 — Mount the EWF image

```bash
mkdir -p /mnt/ewf
ewfmount /path/to/image1.E01 /mnt/ewf/
```

**Check:** `ls /mnt/ewf/` shows `ewf1`; `fdisk -l /mnt/ewf/ewf1` shows the
partition table.

#### Step 2 — Create the loop devices

```bash
losetup -Prf /mnt/ewf/ewf1
losetup -l | grep ewf1
lsblk | grep loop
```

**Check:** `fdisk -l /dev/loopXX` matches the partition table from step 1.

#### Step 3 — Scan and activate the volume group

```bash
vgscan
vgchange -ay vg1
# or, if a PV is missing:
vgchange -ay vg1 --activationmode partial
```

**Check:**

```bash
vgs                # lists the volume groups
lvs                # lists the logical volumes with size and status
lsblk /dev/mapper/ # the device-mapper nodes must have appeared
```

#### Step 4 — Identify the filesystem

Never skip this — it decides the mount command.

```bash
blkid /dev/mapper/vg1-lv1
blkid /dev/mapper/vg1-lv2
```

#### Step 5a — ext4 with a dirty journal

The most common case in forensics.

```bash
mkdir -p /mnt/lv1 /mnt/lv2

mount -t ext4 -o ro,noload /dev/mapper/vg1-lv1 /mnt/lv1
mount -t ext4 -o ro,noload /dev/mapper/vg1-lv2 /mnt/lv2
```

> `noload` skips the uncommitted journal and avoids any write. Some files may
> be in an intermediate state — normal for an acquisition taken from a running
> system.

#### Step 5b — XFS with a dirty journal

```bash
mount -t xfs -o ro,norecovery /dev/mapper/vg1-lv1 /mnt/lv1
```

#### Step 5c — LUKS-encrypted logical volume

```bash
cryptsetup luksDump /dev/mapper/vg1-lv2
cryptsetup luksOpen /dev/mapper/vg1-lv2 lv2_dec
mount -o ro /dev/mapper/lv2_dec /mnt/lv2
```

**Post-mount check:**

```bash
df -h | grep /mnt/lv   # mountpoints and sizes
ls /mnt/lv1            # the filesystem tree must be visible
ls /mnt/lv2
```

#### Step 6 — Analysis

```bash
ls -la /mnt/lv2/
find /mnt/lv2/ -name "*.log" 2>/dev/null
```

> Mount `/proc` and `/sys` only if you really need a chroot — and never chroot
> into a forensic acquisition on a production system.

#### Step 7 — Unmount

See [§10](#10-unmounting-and-cleanup) for the full teardown.

---

## 7. Encrypted Volumes

### 7.1 LUKS

```bash
# 1. Confirm the container type
blkid /dev/mapper/vg1-lv2      # → TYPE="crypto_LUKS"

# 2. Inspect header, cipher and key slots
cryptsetup luksDump /dev/mapper/vg1-lv2

# 3. Open it — 'lv2_dec' is the name of the decrypted mapper device
cryptsetup luksOpen /dev/mapper/vg1-lv2 lv2_dec

# 4. Mount the decrypted device read-only
mount -o ro /dev/mapper/lv2_dec /mnt/lv2

# 5. Close it when done (after unmounting)
umount /mnt/lv2
cryptsetup luksClose lv2_dec
```

If `luksOpen` fails, the passphrase is wrong or the master key is missing.
Tools such as `libluksde` can help when you hold the FVEK/master key rather
than a passphrase.

### 7.2 BitLocker with dislocker

`dislocker` exposes a BitLocker volume as a decrypted virtual file
(`dislocker-file`) that you then mount as a normal filesystem — a two-stage
mount.

```bash
sudo apt install dislocker

sudo mkdir -p /mnt/bitlocker /mnt/decrypted

# Decrypt: -r = read-only, -k = key/recovery file
sudo dislocker-fuse -V /mnt/ewf/ewf1 -r -k <BITLOCKER_KEY> -- /mnt/bitlocker

# /mnt/bitlocker now contains 'dislocker-file' — mount it as NTFS
sudo mount -o ro,loop /mnt/bitlocker/dislocker-file /mnt/decrypted
```

Unlock options:

| Flag | Credential                    |
| ---- | ----------------------------- |
| `-u` | User password                 |
| `-p` | Recovery password (48 digits) |
| `-k` | `.BEK` key file               |
| `-f` | FVEK file                     |
| `-r` | Read-only                     |

Teardown, in order:

```bash
sudo umount /mnt/decrypted
sudo umount /mnt/bitlocker     # or: fusermount -u /mnt/bitlocker
```

---

## 8. VMDK Virtual Disks

### 8.1 Method 1 — qemu-nbd (recommended)

Works with every VMDK variant — and with QCOW2, VDI and VHD as well.

```bash
# 1. Install
sudo apt install qemu-utils   # Debian/Ubuntu
sudo dnf install qemu-img     # Fedora/RHEL

# 2. Load the nbd module with partition support
sudo modprobe nbd max_part=8

# 3. Attach the VMDK to an nbd device
sudo qemu-nbd --connect=/dev/nbd0 /path/to/disk1.vmdk

# Read-only attach (recommended on evidence)
sudo qemu-nbd --read-only --connect=/dev/nbd0 /path/to/disk1.vmdk

# 4. Check the detected partitions
sudo fdisk -l /dev/nbd0
lsblk /dev/nbd0

# 5. Mount the partition you want (p1 = first partition)
sudo mkdir -p /mnt/vmdk
sudo mount /dev/nbd0p1 /mnt/vmdk

# 6. When finished: unmount and disconnect
sudo umount /mnt/vmdk
sudo qemu-nbd --disconnect /dev/nbd0
sudo rmmod nbd
```

If the VMDK contains LVM, run the [§5](#5-lvm-volumes) procedure against
`/dev/nbd0pN` after step 4.

For a split VMDK (`disk1-s001.vmdk`, `disk1-s002.vmdk`, …) point `qemu-nbd` at
the small **descriptor** file, `disk1.vmdk`, not at the extents.

### 8.2 Method 2 — vmware-mount

Only available where VMware Workstation/Player is installed.

```bash
sudo vmware-mount /path/to/disk1.vmdk /mnt/vmdk

# Unmount
sudo vmware-mount -d /mnt/vmdk
```

### 8.3 Method 3 — guestmount (libguestfs)

Filesystem-aware: `-i` inspects the image and mounts the guest's partitions at
the right place on its own, LVM included. Excellent for Linux guest VMDKs.

```bash
sudo apt install libguestfs-tools

# Mount read-only, letting libguestfs work out the layout
sudo guestmount -a /path/to/disk1.vmdk -i --ro /mnt/vmdk

# Unmount
sudo guestunmount /mnt/vmdk
```

---

## 9. Proxmox `.vma` Backups

A `.vma` file is the archive format produced by Proxmox VE backups. It bundles
the VM configuration together with all of its disk images. Extract the disks
first, then treat them as ordinary raw images ([§4](#4-mounting-raw-images) and
[§5](#5-lvm-volumes)).

### 9.1 Getting the vma tool

The easiest route outside a Proxmox host is the `ganehag/pve-vma-docker`
image, which ships the `vma` binary:

- https://hub.docker.com/r/ganehag/pve-vma-docker

```bash
docker pull ganehag/pve-vma-docker
```

Run it with the backup folder bind-mounted into `/backup`:

```bash
docker run -it -v <path_to_the_vma_backup_folder>:/backup ganehag/pve-vma-docker:latest
```

```
root@a1b2c3d4e5f6:/backup#
```

The shell opens directly inside the mounted backup folder. When extracting,
remember to use a `./<folder>` target path so the output lands in the
bind-mounted directory and therefore stays on the host.

### 9.2 Listing the archive contents

```bash
vma list file1.vma
```

```
CFG: size: 854 name: qemu-server.conf
DEV: dev_id=1 size: 131072 devname: drive-efidisk0
DEV: dev_id=2 size: 10737418240 devname: drive-scsi0
DEV: dev_id=3 size: 107374182400 devname: drive-scsi1
CTIME: Mon Jan 01 10:00:00 2024
```

- `CFG` — the VM configuration file (`qemu-server.conf`).
- `DEV` — one line per disk, with its size in bytes and its device name.
- `CTIME` — when the backup was taken.

### 9.3 Extracting the disk images

```bash
vma extract -v file1.vma ./<target_dir>
```

> **Note:** always choose a target directory that lives on the bind-mounted
> volume, i.e. outside the container's own filesystem. Otherwise the extracted
> images disappear with the container — and they are large.

The extraction produces one raw image per disk, named after the device:

```
disk-drive-efidisk0.raw
disk-drive-scsi0.raw     ← usually the OS disk
disk-drive-scsi1.raw     ← usually the data disk (often LVM)
```

From here:

- **OS disk with plain partitions** → [§4.1 offset mount](#41-method-a--mount-with-an-offset)
- **Data disk with LVM** → [§5.2 activate and mount](#52-activating-and-mounting-a-logical-volume)

---

## 10. Unmounting and Cleanup

### 10.1 Teardown order

Always unwind the stack in the **reverse** order in which you built it.
Skipping a layer leaves stale device-mapper nodes and loop devices behind, and
the next image you attach will fail in confusing ways.

```
umount the filesystems
   → close the encrypted containers (luksClose / dislocker)
      → deactivate the volume groups (vgchange -an)
         → release the loop / nbd devices (losetup -d, qemu-nbd -d, kpartx -d)
            → unmount the EWF container (umount or fusermount -u)
```

### 10.2 Step-by-step procedure

**Step 0 — See what is currently mounted**

```bash
mount | grep loop
lsblk
df -h | grep /mnt
```

**Step 1 — Unmount the filesystems**

```bash
sudo umount /mnt/lv2
sudo umount /mnt/lv1
sudo umount /mnt/vmroot
```

> `target is busy`? Find the processes holding the mountpoint open with
> `lsof +D /mnt/lv2` (or `fuser -vm /mnt/lv2`), close them, then retry. Make
> sure your own shell is not sitting inside the directory.

**Step 2 — Close any encrypted container**

```bash
sudo cryptsetup luksClose lv2_dec
sudo umount /mnt/bitlocker        # or fusermount -u /mnt/bitlocker
```

**Step 3 — Deactivate the logical volumes**

```bash
sudo vgchange -an vg1

# or deactivate every volume group:
sudo vgchange -an
```

**Check:** in `lvs` the `Attr` field must go from `-wi-a-----` back to
`-wi-------`.

**Step 4 — Release the loop / nbd devices**

```bash
# Find the loop attached to the image
losetup -l | grep ewf1

# Detach a single loop device
sudo losetup -d /dev/loopXX

# Detach every loop device (careful on a shared machine)
sudo losetup -D

# kpartx mappings
sudo kpartx -dv /mnt/ewf/ewf1

# nbd devices
sudo qemu-nbd --disconnect /dev/nbd0
sudo rmmod nbd
```

**Step 5 — Unmount the EWF container**

```bash
sudo umount /mnt/ewf

# if umount refuses (it is a FUSE mount):
sudo fusermount -u /mnt/ewf
```

### 10.3 Final verification

```bash
losetup -l          # the image's loop device must be gone
lsblk               # no leftover LVM device-mapper nodes
lvs                 # no active volumes
mount | grep loop   # nothing left mounted
ls /mnt/ewf/        # empty directory
```

Once all five commands come back clean, the filesystems are unmounted, the LVM
volumes deactivated and the image safely released.

---

## 11. Troubleshooting

| Error                                          | Cause                                                              | Fix                                                                             |
| ---------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| `unknown filesystem type 'LVM2_member'`        | Trying to mount a raw LVM partition                                | Activate the VG first with `vgchange -ay`, then mount the LV                    |
| `wrong fs type, bad option, bad superblock`    | Dirty journal on ext4/XFS, or wrong offset                         | Use `-o ro,noload` (ext4) or `-o ro,norecovery` (XFS); re-check the byte offset |
| `Volume group "X" not found`                   | Typo in the VG name                                                | Copy the exact name from `vgscan` — it is case-sensitive                        |
| `/dev/vg1 is not a block device`               | Trying to mount the VG instead of the LV                           | Specify the LV: `/dev/vg1/lv1` or `/dev/mapper/vg1-lv1`                         |
| `cannot mount read-only`                       | The device is read-only and the kernel wants to replay the journal | Add `-o ro,norecovery` (or `noload` for ext3/ext4) explicitly                   |
| `target is busy` on umount                     | A process still has files open there                               | `lsof +D /mnt/point` or `fuser -vm /mnt/point`, terminate, retry                |
| `WARNING: Couldn't find device with uuid …`    | The VG spans a PV you do not have                                  | `vgchange -ay vg1 --activationmode partial`                                     |
| `luksOpen` fails                               | Wrong passphrase, or you hold the FVEK rather than a passphrase    | Verify the key, or use a tool such as `libluksde`                               |
| `losetup: could not find any free loop device` | All loop devices are in use                                        | `losetup -D` to clean up, or `modprobe loop max_loop=64`                        |
| `ewfmount` shows an empty directory            | The mount silently failed, or the segment set is incomplete        | Check `dmesg`, confirm every `.E0x` segment is present next to `.E01`           |
| `qemu-nbd` hangs or `/dev/nbd0` is missing     | The `nbd` module was not loaded with partition support             | `sudo modprobe nbd max_part=8`                                                  |
| `mount: /dev/loop0p1 does not exist`           | The partition table was not scanned                                | Re-attach with `losetup -P`, or use `kpartx -av`                                |

---

## 12. Mount Options Quick Reference

| Option                      | Effect                                                     |
| --------------------------- | ---------------------------------------------------------- |
| `ro`                        | Mount read-only — the default choice for any evidence      |
| `loop`                      | Mount a file directly, setting up a loop device implicitly |
| `offset=N`                  | Start the filesystem N **bytes** into the file             |
| `sizelimit=N`               | Only use the first N bytes of the file                     |
| `noload`                    | **ext3/ext4** — do not load or replay the journal          |
| `norecovery`                | **XFS** (also accepted by ext4) — skip log recovery        |
| `noexec`                    | Forbid execution of binaries from the mount                |
| `nodev`                     | Ignore device files on the mount                           |
| `nosuid`                    | Ignore setuid/setgid bits                                  |
| `show_sys_files`            | **NTFS-3G** — expose `$MFT` and the other metadata files   |
| `streams_interface=windows` | **NTFS-3G** — expose alternate data streams                |

A hardened read-only mount for untrusted evidence:

```bash
sudo mount -o ro,noload,noexec,nodev,nosuid /dev/mapper/vg1-lv1 /mnt/lv1
```

---

## 13. Flow Diagrams

**LVM inside an EWF image**

```
ewfmount image1.E01 /mnt/ewf/
        │
        ▼
losetup -Prf /mnt/ewf/ewf1   →  /dev/loopXXp1 (boot)
                                /dev/loopXXp2 (LVM PV)
        │
        ▼
vgscan + vgchange -ay vg1    →  /dev/mapper/vg1-lv1  (root)
                                /dev/mapper/vg1-lv2  (home)
                                /dev/mapper/vg1-lv3  (swap)
        │
        ▼
blkid → filesystem type
        │
        ├─ ext4  →  mount -t ext4 -o ro,noload
        ├─ xfs   →  mount -t xfs  -o ro,norecovery
        └─ LUKS  →  cryptsetup luksOpen → mount -o ro

--- FORENSIC ANALYSIS ---

        │  (unmount in reverse order)
        ▼
umount /mnt/lv*
vgchange -an vg1
losetup -d /dev/loopXX
umount /mnt/ewf
```

**Choosing an entry point by image format**

```
        What do you have?
                │
   ┌────────────┼──────────────┬───────────────┐
   ▼            ▼              ▼               ▼
 .raw/.dd     .E01/EWF      .vmdk           .vma
   │            │              │               │
   │            │              │               ▼
   │            │              │        vma extract → .raw
   │            │              │               │
   │            ▼              ▼               │
   │       ewfmount      qemu-nbd /            │
   │            │        guestmount            │
   │            │              │               │
   └────────────┴──────┬───────┴───────────────┘
                       ▼
              losetup -Pf / kpartx
                       │
                       ▼
                     blkid
                       │
        ┌──────────────┼───────────────┐
        ▼              ▼               ▼
   plain fs        LVM2_member     crypto_LUKS /
        │              │            BitLocker
        │              ▼               │
        │        vgchange -ay          ▼
        │              │        cryptsetup luksOpen /
        │              │        dislocker-fuse
        │              │               │
        └──────────────┴───────────────┘
                       ▼
                mount -o ro,…
```
