# Storage

## The LVM Hierarchy: How It Works

#### Physical Storage Device
1. Physical Volume (PV)
2. Volume Group (VG)  (The shared pool of storage)
3. Logical Volume (LV) (Acts like a virtual partition)
4. Formatted Filesystem (XFS/ext4) & Mounted Directory

#### 1. Initialize the Physical Volume (PV)

Select the raw disk or partition you want to use (for example, /dev/nvme1n1 or /dev/sdb) and convert it into an LVM Physical Volume:

'''
# Create the Physical Volume
sudo pvcreate /dev/nvme1n1

# Verify creation
sudo pvs
'''