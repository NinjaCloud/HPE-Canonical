## Initialization
```
lxd init
```

## Container Management
```
lxc list
lxc launch <image> <container-name>
lxc stop <container-name>
lxc start <container-name>
lxc delete <container-name>
lxc info <container-name>
```

## Network Management
```
lxc network list
lxc network create <network-name>
lxc network delete <network-name>
lxc network attach <network-name> <container-name> <device-name>
```

## Storage Management
```
lxc storage list
lxc storage create <pool-name> <driver>
lxc storage delete <pool-name>
lxc storage volume attach <pool-name> <volume-name> <container-name> <path>
```

## Profiles Management
```
lxc profile list
lxc profile create <profile-name>
lxc profile delete <profile-name>
lxc profile apply <container-name> <profile1> <profile2> ...
lxc profile show <profile-name>
lxc profile set <profile-name> <key> <value>
```

## Snapshots
```
lxc snapshot list <container-name>
lxc snapshot <container-name> <snapshot-name>
lxc restore <container-name> <snapshot-name>
lxc delete <container-name>/<snapshot-name>
```

## File Management
```
lxc file push <local-path> <container-name>/<container-path>
lxc file pull <container-name>/<container-path> <local-path>
```

## Exec Commands
```
lxc exec <container-name> -- <command>
lxc exec <container-name> -- /bin/bash
```

## Miscellaneous
```
lxc version
lxc config show
lxc config set <key> <value>
lxc import <backup-file>
```
