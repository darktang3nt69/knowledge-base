[If you're unable to find the **Add Directory** option in the Proxmox GUI, it may be because the interface options have slightly changed, or the permissions/settings might not be fully enabled. Here’s an alternative way to add a storage directory in Proxmox, specifically from the command line, which should accomplish the same result.

### Step-by-Step: Adding a Storage Directory on Proxmox via CLI

1. **Create the Directory**:
   - SSH into your Proxmox server.
   - Create a directory on your NVMe drive where you want to store your VMs, containers, and shared data. For example:
     ```bash
     mkdir -p /mnt/nvme_storage
     ```

2. **Add the Directory as Proxmox Storage**:
   - Use the following command to add the storage configuration:
     ```bash
     pvesm add dir nvme_storage --path /mnt/nvme_storage --content images,rootdir,iso,vztmpl,backup
     ```
   - Here’s what each `--content` option means:
     - `images`: VM disk images
     - `rootdir`: LXC container root directories
     - `iso`: ISO images
     - `vztmpl`: LXC templates
     - `backup`: Backup files

3. **Verify the Storage Addition**:
   - Go back to the Proxmox GUI and navigate to **Datacenter > Storage**. You should now see `nvme_storage` listed as a storage option.
   - You can manage VMs, LXC containers, and backups using this new directory.

4. **Set Permissions (Optional)**:
   - To ensure the directory is accessible, apply full permissions:
     ```bash
     chmod -R 777 /mnt/nvme_storage
     ```

This approach should allow you to add the storage directory without needing to use the GUI, achieving the same result directly through the command line. Let me know if this works for you!](<To configure your 200GB partition on `/dev/nvme0n1p1` as an NFS path, here’s a detailed guide in Obsidian format.

---

# Configuring a 200GB Partition as an NFS Path on Orange Pi

This guide explains how to set up your 200GB NVMe partition as an NFS (Network File System) share so you can use it with your LXC containers and VMs.

---

## Steps

### 1. Format the Partition

Format the 200GB partition (`/dev/nvme0n1p1`) with a Linux filesystem such as `ext4`.

```bash
sudo mkfs.ext4 /dev/nvme0n1p1
```

### 2. Create a Mount Point

Create a directory where you want to mount the partition. This will act as your NFS share root.

```bash
sudo mkdir -p /mnt/nfs_share
```

### 3. Mount the Partition

Mount the partition to your newly created directory.

```bash
sudo mount /dev/nvme0n1p1 /mnt/nfs_share
```

To make this mount persistent across reboots, add an entry in `/etc/fstab`.

```plaintext
/dev/nvme0n1p1  /mnt/nfs_share  ext4  defaults  0  2
```

### 4. Install NFS Server

Install the NFS server package if it’s not already installed.

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```

### 5. Configure the NFS Export

Edit the NFS exports file to share the directory.

```bash
sudo nano /etc/exports
```

Add the following line to export the `/mnt/nfs_share` directory:

```plaintext
/mnt/nfs_share  *(rw,sync,no_subtree_check,no_root_squash)
```

Explanation of options:

- `rw`: Read and write access.
- `sync`: Ensures changes are written immediately.
- `no_subtree_check`: Disables subtree checking for better performance.
- `no_root_squash`: Allows root users to have full access.

%3E **Note**: Replace `*` with a specific IP or subnet (e.g., `192.168.1.0/24`) if you want to limit access.

### 6. Export the Share

Reload the NFS exports to apply changes:

```bash
sudo exportfs -arv
```

### 7. Start and Enable the NFS Service

Ensure the NFS service starts on boot:

```bash
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

### 8. Verify the NFS Export

You can check the exported shares with:

```bash
sudo exportfs -v
```

---

## Accessing the NFS Share from Clients

On client systems (e.g., LXC containers or VMs), mount the NFS share.

1. Install the NFS client (on the client machine):

   ```bash
   sudo apt install nfs-common -y
   ```

2. Create a mount point on the client, then mount the NFS share:

   ```bash
   sudo mkdir -p /mnt/nfs_client
   sudo mount %3Cserver_ip>:/mnt/nfs_share /mnt/nfs_client
   ```

   Replace `<server_ip>` with the IP address of your Orange Pi host.

3. To mount the NFS share automatically on boot, add the following entry in `/etc/fstab` on the client machine:

   ```plaintext
   <server_ip>:/mnt/nfs_share  /mnt/nfs_client  nfs  defaults  0  0
   ```

---

With this setup, your 200GB NVMe partition should now be accessible as an NFS share across your network.>)