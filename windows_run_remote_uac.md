# Windows Remote UAC Runner

A small Windows helper script for deploying UAC to a remote Unix-like host, mounting remote storage, running UAC on the remote host, and unmounting the storage when finished.

## File

- `windows_run_remote_uac.py` — Windows Python helper script.

## Requirements

- Windows with Python 3.8+ installed.
- `ssh` and `scp` available on Windows (OpenSSH client in PATH).
- Remote endpoint reachable by SSH.
- Remote endpoint able to mount the requested storage source and write output to the mount point.

## Basic usage

The recommended way to run this script is with a configuration file. All required options can be defined in the file, and the command line only needs to pass `--config`.

```powershell
python windows_run_remote_uac.py --config windows_run_remote_uac.example.ini
```

The script also supports command-line overrides when needed, but that is optional.

## Configuration file support

The script supports loading defaults from a configuration file. Values provided on the command line always override values from the config file.

Supported formats:

- JSON
- INI

### Example INI config

```ini
[uac]
remote_host = 10.0.0.1
remote_user = root
remote_port = 22
ssh_options = StrictHostKeyChecking=no,UserKnownHostsFile=/dev/null
local_uac_path = C:\workspaces\uac
remote_temp_root = /tmp/uac_remote
mount_source = //fileserver/share_5g7b2
mount_point = /mnt/output
mount_fstype = cifs
mount_options = username=root,password=Aa123456,uid=0
use_sudo = false
cleanup = true
verbose = true
no_umount_on_error = false
uac_args = --profile ir_triage
```

### Example JSON config

```json
{
  "remote_host": "10.0.0.1",
  "remote_user": "root",
  "remote_port": 22,
  "ssh_options": ["StrictHostKeyChecking=no", "UserKnownHostsFile=/dev/null"],
  "local_uac_path": "C:\\workspaces\\uac",
  "remote_temp_root": "/tmp/uac_remote",
  "mount_source": "//fileserver/share_5g7b2",
  "mount_point": "/mnt/output",
  "mount_fstype": "cifs",
  "mount_options": "username=root,password=Aa123456,uid=0",
  "use_sudo": false,
  "cleanup": true,
  "verbose": true,
  "no_umount_on_error": false,
  "uac_args": ["--profile", "ir_triage"]
}
```

### Run with a config file

```powershell
python windows_run_remote_uac.py --config windows_run_remote_uac.example.ini
```

### Run with config plus command-line override

```powershell
python windows_run_remote_uac.py --config windows_run_remote_uac.example.ini --mount-source //override/share --verbose -- --profile ir_triage
```

## Command-line options

- `--config` — Path to a JSON or INI configuration file.
- `--remote-host` — Remote SSH host or IP address.
- `--remote-user` — Remote SSH username.
- `--remote-port` — Remote SSH port.
- `--ssh-option` — Additional SSH option to pass to `ssh` and `scp`.
- `--local-uac-path` — Local path to the extracted UAC directory.
- `--remote-temp-root` — Remote base directory for the temporary UAC deployment.
- `--remote-temp-name` — Optional remote temporary directory name.
- `--mount-source` — Remote mount source (network share, device, path).
- `--mount-point` — Remote mount target directory.
- `--mount-fstype` — Filesystem type for the remote mount.
- `--mount-options` — Options for the remote mount command.
- `--mount-command` — Custom mount command to run instead of the default `mount` invocation.
- `--use-sudo` — Use `sudo` for remote mount and unmount operations.
- `--cleanup` — Remove the temporary remote UAC directory after execution.
- `--verbose` — Print verbose progress and commands.
- `--no-umount-on-error` — Skip unmount on remote UAC execution failure.
- `--uac-args` — Remaining arguments to pass through to the remote UAC invocation.

## Behavior

1. Copy the local UAC directory to a generated remote temporary directory.
2. Mount the configured remote storage on the remote endpoint.
3. Run the remote `uac` binary with the mount point as the output destination.
4. Unmount the storage when UAC completes.
5. Optionally clean up the temporary remote UAC directory.

## Notes

- The remote host must provide `mount` and `umount`.
- Use `--mount-command` when a custom mount invocation is required.
- Use `--use-sudo` when mount/unmount requires elevated privileges.
- `ssh_options` may be set as a comma-separated string in INI or as a list in JSON.
