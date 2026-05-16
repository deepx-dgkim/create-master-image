# Raspberry Pi SD Card Backup Guide (PiShrink)

This guide explains how to create a minimized Raspberry Pi SD card backup image using `dd` and `PiShrink`.

The resulting `.img` or `.img.gz` file size will be close to the actual used filesystem size instead of the full SD card capacity.

Example:

* SD card size: 128GB
* Actual used space: 30GB
* Final compressed image: ~8GB–20GB (depends on data)

---

# Requirements

* Linux PC (Ubuntu/Debian recommended)
* SD card reader
* Raspberry Pi SD card
* Sufficient disk space

---

# 1. Insert SD Card

Insert the Raspberry Pi SD card into your Linux PC.

Check device name:

```bash
lsblk
```

Example output:

```bash
sda      931.5G
sdb      119.2G
├─sdb1     512M
└─sdb2   118.7G
```

In this example:

```text
SD card device = /dev/sdb
```

> WARNING:
> Make sure you select the correct device.
> Using the wrong device may overwrite your system disk.

---

# 2. Check Filesystem (Recommended)

Run filesystem check before backup:

```bash
sudo e2fsck -f /dev/sdb2
```

---

# 3. Create Raw Image

Create full raw image using `dd`:

```bash
sudo dd if=/dev/sdb of=raspi_backup.img bs=64M status=progress conv=fsync
```

Parameters:

| Option            | Description         |
| ----------------- | ------------------- |
| `if=`             | Input device        |
| `of=`             | Output image file   |
| `bs=64M`          | Faster block size   |
| `status=progress` | Show progress       |
| `conv=fsync`      | Flush writes safely |

---

# 4. Install PiShrink

Clone PiShrink repository:

```bash
git clone https://github.com/Drewsif/PiShrink.git
cd PiShrink
chmod +x pishrink.sh
```

Official repository:

[PiShrink GitHub](https://github.com/Drewsif/PiShrink?utm_source=chatgpt.com)

---

# 5. Shrink Image

Shrink image without compression:

```bash
sudo ./pishrink.sh ../raspi_backup.img
```

Or shrink + gzip compress:

```bash
sudo ./pishrink.sh -z ../raspi_backup.img
```

Result:

```text
raspi_backup.img.gz
```

---

# What PiShrink Does

PiShrink automatically:

1. Shrinks ext4 filesystem
2. Resizes partition table
3. Removes unused blocks
4. Truncates image size
5. Optionally compresses image

This significantly reduces image size.

---

# 6. Restore Image to SD Card

Restore compressed image:

```bash
gunzip -c raspi_backup.img.gz | sudo dd of=/dev/sdb bs=64M status=progress conv=fsync
```

Or restore uncompressed image:

```bash
sudo dd if=raspi_backup.img of=/dev/sdb bs=64M status=progress conv=fsync
```

---

# Automatic Filesystem Expansion

Most Raspberry Pi OS images automatically expand the filesystem during first boot.

This means:

* Small minimized image
* Full SD card capacity restored automatically after boot

---

# Useful Tips

## Show Disk Usage

Check image size:

```bash
du -sh raspi_backup.img*
```

---

## Better Compression

For higher compression ratio:

```bash
xz -T0 -z raspi_backup.img
```

This is slower but produces smaller files than gzip.

---

# Recommended Workflow

```text
SD Card
   ↓
dd backup
   ↓
PiShrink
   ↓
Compressed minimal image
```

---

# Notes

* PiShrink mainly supports ext4-based Raspberry Pi OS images.
* Always safely unmount SD cards before removal.
* Avoid creating images from mounted/read-write partitions when possible.

---

# References

* [PiShrink GitHub](https://github.com/Drewsif/PiShrink?utm_source=chatgpt.com)
* [Raspberry Pi Official Website](https://www.raspberrypi.com/?utm_source=chatgpt.com)

