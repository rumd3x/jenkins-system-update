# jenkins-system-update

A Jenkins declarative pipeline that performs a non-interactive `apt` update and upgrade on a remote Debian-based host over SSH, and reboots the machine if the system requires it.

## Requirements

On the **Jenkins agent** (label `docker`):
- `sshpass`

On the **target host**:
- SSH access with a user that has `sudo` privileges
- `update-notifier-common` (provides `/var/run/reboot-required`; installed by default on Ubuntu)

## Parameters

| Parameter | Type | Description |
|---|---|---|
| `HOST` | String | Hostname or IP address of the target machine |
| `SSH_CREDENTIALS` | Credentials | Username + password credential stored in Jenkins |

## What it does

1. **Update packages** — SSHes into `HOST` and runs:
   ```
   apt-get update
   apt-get upgrade -y    # DEBIAN_FRONTEND=noninteractive
   ```
2. **Reboot if required** — checks for `/var/run/reboot-required`; if present, issues `sudo reboot now` and lets the connection drop naturally.

## Credentials setup

In Jenkins → **Manage Jenkins → Credentials**, add a **Username with password** credential and use its ID as the `SSH_CREDENTIALS` parameter value.

## Security notes

- The password is passed to `sshpass` via the `SSHPASS` environment variable (`sshpass -e`) rather than as a CLI argument, so it does not appear in the process list.
- `sudo -S` reads the password from stdin; it is never written to disk or echoed to the console.
- Jenkins masks the credential values in the build log automatically via `withCredentials`.

## License

[MIT](LICENSE)
