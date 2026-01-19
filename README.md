# This is PVE API (not all of them) for project references

## Core infrastructure endpoints
| Area          | Method & Path              | Purpose              | Key details                                                                            |
| ------------- | -------------------------- | -------------------- | -------------------------------------------------------------------------------------- |
| API index     | GET /                      | Top‑level API tree   | Lists main collections: access, cluster, nodes, pools, storage, version.pve.proxmox+1​ |
| Version       | GET /version               | PVE version info     | Returns API and product version; useful for compatibility checks.pve.proxmox​          |
| Nodes list    | GET /nodes                 | List cluster nodes   | Includes node name, status, uptime, and basic resource info.pve.proxmox+1​             |
| Node status   | GET /nodes/{node}/status   | Detailed node status | CPU, memory, KSM, load, rootfs, and overall node health.pve.proxmox+1​                 |
| Node services | GET /nodes/{node}/services | List system services | Supports `POST /nodes/{node}/services/{service}/start                                  |

## QEMU virtual machine endpoints
| Area                 | Method & Path                                           | Purpose                          | Key details                                                                                                                                       |
| -------------------- | ------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| List VMs on node     | GET /nodes/{node}/qemu                                  | Enumerate VMs on a node          | Returns VMID, name, status, CPU, memory, and node.publicapi+1​                                                                                    |
| Create VM            | POST /nodes/{node}/qemu                                 | Create new VM                    | Accepts vmid, name, memory, cores, sockets, disks (scsi0, ide0 etc.), net0, ostype, and storage options; returns UPID for async task.publicapi+1​ |
| VM config            | GET /nodes/{node}/qemu/{vmid}/config                    | Get VM configuration             | Full config: CPU, memory, disks, networks, flags; used by panels to render edit forms.pve.proxmox​                                                |
| Update config        | POST /nodes/{node}/qemu/{vmid}/config                   | Change VM config                 | Many fields are hot‑pluggable (e.g. memory balloon, vCPU) but not all; treat as async task.pve.proxmox​                                           |
| Status – start       | POST /nodes/{node}/qemu/{vmid}/status/start             | Power on VM                      | Optional params: force, skiplock; returns UPID.pve.proxmox+1​                                                                                     |
| Status – stop        | POST /nodes/{node}/qemu/{vmid}/status/stop              | ACPI shutdown / hard stop        | Parameters for graceful vs. immediate stop; important for automation safety.pve.proxmox+1​                                                        |
| Status – reboot      | POST /nodes/{node}/qemu/{vmid}/status/reboot            | Reboot guest                     | Uses ACPI; may fail if guest does not respond.pve.proxmox​                                                                                        |
| Status – reset       | POST /nodes/{node}/qemu/{vmid}/status/reset             | Hard reset                       | Equivalent to power‑cycle at hypervisor level.pve.proxmox​                                                                                        |
| Pause / resume       | `POST /nodes/{node}/qemu/{vmid}/status/pause            | resume`                          | Suspend/resume execution                                                                                                                          |
| Delete VM            | DELETE /nodes/{node}/qemu/{vmid}                        | Remove VM definition and disks   | Options to keep disks or remove all storage; returns UPID.pve.proxmox​                                                                            |
| Console / VNC proxy  | POST /nodes/{node}/qemu/{vmid}/vncproxy                 | Get VNC ticket                   | Returns port, ticket, user, password for noVNC/SPICE frontends.pve.proxmox​                                                                       |
| Cloud‑init           | GET/POST /nodes/{node}/qemu/{vmid}/cloudinit/dump       | Manage cloud‑init data           | Used to inject user‑data/meta‑data, network config via drive ide2/scsiX depending on template.pve.proxmox​                                        |
| Snapshots – list     | GET /nodes/{node}/qemu/{vmid}/snapshot                  | List VM snapshots                | Returns tree of snapshots with parent/child relationships.pve.proxmox​                                                                            |
| Snapshots – create   | POST /nodes/{node}/qemu/{vmid}/snapshot                 | Create snapshot                  | Supports live snapshots for supported storage; can include RAM for running guests.pve.proxmox​                                                    |
| Snapshots – rollback | POST /nodes/{node}/qemu/{vmid}/snapshot/{snap}/rollback | Revert to snapshot               | May require VM to be stopped depending on snapshot type.pve.proxmox​                                                                              |
| Clone                | POST /nodes/{node}/qemu/{vmid}/clone                    | Clone VM/template                | Supports full/linked clone, target storage, and target node; returns new VMID.pve.proxmox​                                                        |
| Migrate              | POST /nodes/{node}/qemu/{vmid}/migrate                  | (Live) migration to another node | Parameters: target, online (live), with-local-disks etc.; blocked if snapshots present.pve.proxmox+1​                                             |
| Agent                | GET /nodes/{node}/qemu/{vmid}/agent/{command}           | QEMU guest agent calls           | Commands like get-osinfo, exec, fsfreeze-freeze etc. if agent enabled in guest.pve.proxmox​                                                       |

## Cluster & global endpoints
| Area                  | Method & Path                    | Purpose                | Key details                                                                                                      |
| --------------------- | -------------------------------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Cluster resources     | GET /cluster/resources           | Global resource view   | Returns nodes, VMs, CTs, storage; supports type filter, but behavior can be quirky for some types.pve.proxmox+1​ |
| Cluster status        | GET /cluster/status              | Corosync/pmxcfs status | Shows nodes, quorum, and membership info.pve.proxmox​                                                            |
| Cluster config        | GET /cluster/config              | Basic cluster config   | Node list, cluster name, ring configuration.pve.proxmox​                                                         |
| HA resources          | GET /cluster/ha/resources        | HA‑managed services    | Lists HA‑enabled VMs/CTs with state (started, stopped, error).pve.proxmox+1​                                     |
| HA groups             | GET /cluster/ha/groups           | HA node groups         | Node sets with priorities for HA placement.pve.proxmox​                                                          |
| HA resources – update | POST /cluster/ha/resources/{sid} | Modify HA resource     | Set state (started/stopped), max_restart, max_relocate etc.pve.proxmox+1​                                        |
| Backup jobs           | GET /cluster/backup              | List vzdump jobs       | Shows schedule, selection mode, compression, target storage.pve.proxmox​                                         |
| Backup jobs – create  | POST /cluster/backup             | Define new backup job  | Supports mode (snapshot, suspend, stop), include/exclude, mail options.pve.proxmox​                              |
| Pools                 | GET /pools                       | List resource pools    | Pools group VMs/CTs/storage for ACL management.pve.proxmox​                                                      |

## Storage & backup endpoints
| Area                 | Method & Path                                | Purpose                       | Key details                                                                                               |
| -------------------- | -------------------------------------------- | ----------------------------- | --------------------------------------------------------------------------------------------------------- |
| Storage list         | GET /nodes/{node}/storage                    | List storages visible on node | Each storage has type, content types, enabled flag, shared flag.pve.proxmox+1​                            |
| Storage content      | GET /nodes/{node}/storage/{storage}/content  | List contents                 | VM disks, CT volumes, ISOs, templates, backups; filterable by content (images, backup, etc.).pve.proxmox​ |
| Upload ISO/template  | POST /nodes/{node}/storage/{storage}/upload  | Upload content                | Used for ISO/backup upload via web UI; multipart/form‑data.pve.proxmox​                                   |
| VZDump backup (node) | POST /nodes/{node}/vzdump                    | Ad‑hoc backup job             | Parameters mirror scheduled jobs: mode, compress, storage, exclude, maxfiles, etc.pve.proxmox​            |
| Restore VM           | POST /nodes/{node}/qemu (with archive, vmid) | Restore from backup           | Effectively qmrestore via API; archive path from storage/.../content listing.pve.proxmox+1​               |
| Restore CT           | POST /nodes/{node}/lxc (with restore=1)      | Restore container             | Similar semantics to pct restore; rootfs target and options via body parameters.pve.proxmox+1​            |

## Firewall endpoints
| Area                        | Method & Path                                       | Purpose                  | Key details                                                                        |
| --------------------------- | --------------------------------------------------- | ------------------------ | ---------------------------------------------------------------------------------- |
| Datacenter firewall options | GET/POST /cluster/firewall/options                  | Global firewall settings | Enable/disable default rules, log levels, policy, NDP, MAC filter.pve.proxmox+1​   |
| Datacenter rules            | GET/POST /cluster/firewall/rules                    | DC‑level rules           | Ordered rules; fields like type, action, iface, source, dest, dport.pve.proxmox+1​ |
| Node firewall options       | GET/POST /nodes/{node}/firewall/options             | Node firewall            | Per‑node enable, logging levels, input/output policy.pve.proxmox+1​                |
| Node rules                  | GET/POST /nodes/{node}/firewall/rules               | Rules on node            | Same rule model as datacenter but limited to specific node.pve.proxmox​            |
| VM firewall options         | GET/POST /nodes/{node}/qemu/{vmid}/firewall/options | Guest firewall toggle    | Enable per‑VM firewall and its default policy.pve.proxmox+1​                       |
| VM rules                    | GET/POST /nodes/{node}/qemu/{vmid}/firewall/rules   | Rules for one VM         | Typically used by panels to expose “firewall” tab per VPS.pve.proxmox+1​           |
| Aliases                     | GET/POST /cluster/firewall/aliases                  | Reusable addresses       | Named IPs/subnets used in rules.pve.proxmox​                                       |
| IP sets                     | GET/POST /cluster/firewall/ipset                    | Bulk address lists       | Scalable list references for rules targeting many IPs.pve.proxmox​                 |

## Access, users, tokens, permissions
| Area             | Method & Path                             | Purpose                 | Key details                                                                                  |
| ---------------- | ----------------------------------------- | ----------------------- | -------------------------------------------------------------------------------------------- |
| Get ticket       | POST /access/ticket                       | Username/password login | Returns ticket and CSRFPreventionToken; used for cookie‑based sessions.pve.proxmox+1​        |
| API token list   | GET /access/users/{user}/token            | List tokens             | Shows existing API tokens for a user.pve.proxmox​                                            |
| API token create | POST /access/users/{user}/token/{tokenid} | Create new token        | Options: expire, privsep; returns token secret once.pve.proxmox​                             |
| Users            | GET/POST/PUT/DELETE /access/users         | Manage users            | Create, modify, disable Proxmox users in realms (pam, pve, etc.).pdm.proxmox+1​              |
| Groups           | GET/POST/DELETE /access/groups            | Group management        | Group resources for shared ACLs.pdm.proxmox+1​                                               |
| Roles            | GET/POST/DELETE /access/roles             | Role definitions        | Built‑ins like Administrator, PVEAdmin, PVEAuditor plus custom privilege sets.pdm.proxmox+1​ |
| ACLs             | GET/POST /access/acl                      | Assign permissions      | Triples of path (e.g. /vms/{vmid}), user/@group, and role.pdm.proxmox+1​                     |

## Tasks, monitoring, misc
| Area            | Method & Path                         | Purpose              | Key details                                                                                |
| --------------- | ------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------ |
| Node tasks list | GET /nodes/{node}/tasks               | Recent tasks on node | Each entry has UPID, type (vzcreate, qmstart, vzdump), status.reddit​                      |
| Task status     | GET /nodes/{node}/tasks/{UPID}/status | Single task result   | status (running/stopped), exitstatus (OK or error), start/end times.reddit​                |
| RRD data        | GET /nodes/{node}/rrd                 | Performance metrics  | Time‑series metrics for CPU, memory, network, etc., selectable via parameters.pve.proxmox​ |
| VM RRD          | GET /nodes/{node}/qemu/{vmid}/rrd     | Per‑VM metrics       | Same as node RRD but scoped to a VM.pve.proxmox​                                           |

