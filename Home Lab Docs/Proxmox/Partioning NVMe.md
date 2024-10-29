
## 1. Delete Existing Partition Table on `/dev/nvme0n1`

1. Open `fdisk` to delete all partitions on `/dev/nvme0n1`:

   ```bash
   sudo fdisk /dev/nvme0n1
   ```

2. Within `fdisk`:
   - Press `d` to delete each partition one at a time.
   - Once all partitions are deleted, press `w` to write the changes and exit.

---

## 2. Re-create Partitions Correctly on `/dev/nvme0n1`

1. Open `fdisk` again on `/dev/nvme0n1`:

   ```bash
   sudo fdisk /dev/nvme0n1
   ```

2. Within `fdisk`:
   - Press `n` to create a new partition.
     - For the **first partition**:
       - Specify the start and end sectors to allocate **200GB**.
     - For the **second partition**:
       - Use the remaining space, approximately **36GB** for the OS.
   - After creating the partitions, press `w` to save the changes.

---

## 3. Verify the Partitions

1. Run `lsblk` to confirm that `/dev/nvme0n1` has been divided into `nvme0n1p1` and `nvme0n1p2`.

   ```bash
   lsblk
   ```

---

## 4. Format the Partitions

1. Format the partitions as needed:

   ```bash
   sudo mkfs.ext4 /dev/nvme0n1p1  # Format for NFS
   sudo mkfs.ext4 /dev/nvme0n1p2  # Format for OS
   ```

---

## 5. Configure `/dev/nvme0n1p1` as NFS

You can now proceed to mount `/dev/nvme0n1p1` as an NFS share and `/dev/nvme0n1p2` for your OS.

---

This will give you a clear, clean partitioning structure on your NVMe drive. Let me know if you encounter any issues during the process!
```

This structure should be easy to navigate and read in Obsidian, with each step broken down and formatted for clarity. Let me know if you need any more details!>)