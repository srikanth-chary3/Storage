# Storage

## The LVM Hierarchy: How It Works

[ Physical Storage Device ]  --->  Physical Volume (PV)
                                        │
                                        ▼
                                 Volume Group (VG)  (The shared pool of storage)
                                        │
                                        ▼
                                Logical Volume (LV) (Acts like a virtual partition)
                                        │
                                        ▼
                               Formatted Filesystem (XFS/ext4) & Mounted Directory

