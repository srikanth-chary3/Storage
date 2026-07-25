# Storage

## The LVM Hierarchy: How It Works

#### Physical Storage Device
1. Physical Volume (PV)
2. Volume Group (VG)  (The shared pool of storage)
3. Logical Volume (LV) (Acts like a virtual partition)
4. Formatted Filesystem (XFS/ext4) & Mounted Directory

#### 1. Initialize the Physical Volume (PV)
Select the raw disk or partition you want to use (for example, /dev/nvme1n1 or /dev/sdb) and convert it into an LVM Physical Volume:
```
# Create the Physical Volume
sudo pvcreate /dev/nvme1n1

# Verify creation
sudo pvs
```

#### 2. Create the Volume Group (VG)
Group one or more Physical Volumes together into a single storage pool called a Volume Group (e.g., naming it DataVG):
```
# Syntax: vgcreate
sudo vgcreate DataVG /dev/nvme1n1

# Verify creation (shows free space in the pool)
sudo vgs
```

#### 3. Create Logical Volumes (LV)
Carve out virtual partitions from your Volume Group pool. You can specify fixed sizes (e.g., -L 20G) or percentages of free space (e.g., -l 100%FREE).
```
# Create a 20GB volume for /var logs
sudo lvcreate -L 20G -n varVol DataVG

# Create a 10GB volume for /app
sudo lvcreate -L 10G -n appVol DataVG

# Use ALL remaining free space for /home
sudo lvcreate -l +100%FREE -n homeVol DataVG

# Verify creation
sudo lvs
```

#### 4. Format the Logical Volumes with a Filesystem
Just like standard partitions, Logical Volumes must be formatted with a filesystem (like XFS or ext4) before Linux can store files on them:
```
# Format as XFS (Default on Amazon Linux / RHEL)
sudo mkfs.xfs /dev/mapper/DataVG-varVol
sudo mkfs.xfs /dev/mapper/DataVG-appVol
sudo mkfs.xfs /dev/mapper/DataVG-homeVol

Device Path Rule: Logical volumes appear in /dev/mapper/<VG_NAME>-<LV_NAME> or /dev/<VG_NAME>/<LV_NAME>.
```

#### 5. Mount the Volumes & Configure Auto-Mount on Reboot
Create target directories and mount your newly formatted volumes:

```
#### 1. Create mount directories
sudo mkdir -p /mnt/var /mnt/app /mnt/home

##### 2. Mount them manually to test
sudo mount /dev/mapper/DataVG-varVol /mnt/var
sudo mount /dev/mapper/DataVG-appVol /mnt/app
sudo mount /dev/mapper/DataVG-homeVol /mnt/home

##### 3. Get UUIDs for /etc/fstab entry
sudo blkid /dev/mapper/DataVG-*
```

* To ensure they stay mounted after a system restart, edit /etc/fstab:
```
sudo nano /etc/fstab

Add entries using the UUIDs returned from blkid:
UUID=YOUR-UUID-VAR  /mnt/var  xfs  defaults,nofail  0  2
UUID=YOUR-UUID-APP  /mnt/app  xfs  defaults,nofail  0  2
UUID=YOUR-UUID-HOME /mnt/home xfs  defaults,nofail  0  2

Test the /etc/fstab entries to ensure there are no syntax errors:
sudo mount -a
```
Phase	                Command	                                Purpose
Physical Volume     	pvcreate /dev/device	                Prepares raw disk for LVM use.
Volume Group	        vgcreate VG_NAME /dev/device	        Creates the central storage pool.
Logical Volume	        lvcreate -L 20G -n LV_NAME VG_NAME	    Carves out virtual partitions from the pool.
Filesystem	            mkfs.xfs /dev/mapper/VG_NAME-LV_NAME	Writes XFS/ext4 structure to the LV.
Mounting	            mount /dev/mapper/... /mount/point	    Connects the storage to a system folder.