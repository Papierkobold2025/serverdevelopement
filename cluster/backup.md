# Backup configuration

Integration of Proxmox Backup Server (PBS) as the fifth dedicated node of the cluster, for centralized automatic backup of all VMs — separated into its own document since it backs up the entire cluster, not just Nextcloud.

## Node specifications (pbs-homelab)
- CPU(s): 8 x Intel Core i7-8559U @ 2.70GHz (1 Socket)
- Total RAM: 62.67 GiB
- Storage (root): 956.93 GB
- Backup datastore (vmbackup-homelab-local): 1.87 TB (current usage: 120.12 GB, 6.43%)
- Kernel: 6.17.2-1-pve
- Product: Proxmox Backup Server (not Proxmox VE)

## Backup Job Configuration

| Field | Value |
|---|---|
| Storage | pbs-homelab |
| Included nodes | All (-- All --) |
| Schedule | 2:30 AM |
| Mode | Snapshot |
| Compression | ZSTD (fast and good) |
| Selection | All VMs |

## Retention

| Type | Amount |
|---|---|
| Daily | 7 |
| Weekly | 4 |

- Approximate coverage: full last month, without indefinite accumulation.

## Verification and integrity
- Verification (Verify) configured to run immediately after each backup, not on a separate schedule.
- Restoration successfully tested on at least one occasion (test VM).

## Notifications
- Success/failure notifications for the job configured by email, reusing the same SMTP configured for Nextcloud (Gmail, port 587, Application Password — see `docs/nextcloud.md`).
