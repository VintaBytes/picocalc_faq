# 📌 How to Clone an SD Card in Ubuntu?

The goal is to create a backup of the original SD card that comes with the PicoCalc, so it can be restored to another SD in case of failure or data loss.

---

## Step 1: Identify the SD device

First, connect the card and list the disks:

```bash
lsblk -o NAME,SIZE,MODEL,TRAN,MOUNTPOINTS
```

This shows all disks, their sizes, and mount points.
In my case, the card appeared as **/dev/sdf**, with two partitions:

```
/dev/sdf1  → 29.5G (FAT32)
/dev/sdf2  → 32M  (Linux)
```

---

## Step 2: Unmount the SD before copying

Make sure to unmount the partitions to avoid errors:

```bash
sudo umount /dev/sdf1
sudo umount /dev/sdf2
```

---

## Step 3: Create the SD image (backup)

We use **ddrescue** instead of `dd`, because it handles read errors better.
If you don’t have it installed, you can add it with:

```bash
sudo apt install gddrescue
```

Once installed, run the following command:

```bash
sudo ddrescue -d -D --force /dev/sdf ~/sdcard_picocalc.img ~/ddrescue.log
```

Where:

* `/dev/sdf` → SD device
* `~/sdcard_picocalc.img` → resulting image file
* `~/ddrescue.log` → log file to continue if errors occur

This generates a full copy of the card in a `.img` file.

---

## Step 4: Write the image to a new SD

If you also want to copy this image to a new card, insert the destination SD and check its name again with `lsblk`.
In my case, it appeared again as **/dev/sdf** (unmounted).

To restore, write the complete image to the device:

```bash
sudo ddrescue -D --force ~/sdcard_picocalc.img /dev/sdf /tmp/ddrescue-write.log
sync
```

This overwrites the entire SD with the exact contents of the original.

---

## Step 5: Verify the copy

Optionally, we can check that the first bytes of the SD match the image:

```bash
IMG=~/sdcard_picocalc.img
SIZE=$(stat -c%s "$IMG")
sudo cmp -n "$SIZE" "$IMG" /dev/sdf && echo "Verified OK"
```

If you see the message **“Verified OK”**, the copy is identical.

---

## Step 6: Safely eject the SD

Finally, eject the SD so that the partition table is refreshed:

```bash
sudo udevadm settle
sudo eject /dev/sdf
```

---

# Result

The new SD card is ready to be used in the PicoCalc, working exactly like the original.
This procedure can be used to **clone any SD card** (Raspberry Pi, retro consoles, etc.).



¿Querés que además lo deje preparado en **PDF en inglés con el texto formateado** para que lo tengas como mini-manual listo para compartir?
