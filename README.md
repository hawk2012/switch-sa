# SELinux to AppArmor Switch Tool

`switch-sa` is a Python-based utility designed to switch between SELinux and AppArmor on Linux systems. It supports major enterprise Linux distributions, including Debian/Ubuntu and RHEL/CentOS/Fedora-based systems.

The tool automates:
- Disabling the currently active MAC (Mandatory Access Control) system.
- Enabling the alternative system.
- Installing required packages if missing.
- Performing a basic cross-conversion of access rules using system logs.
- Preparing the system for relabeling (SELinux) or reloading profiles (AppArmor).

This is particularly useful during system hardening transitions, policy migration, or evaluation of alternative MAC frameworks.

## Features

- Auto-detects the Linux distribution (Debian/Ubuntu vs RHEL-family).
- Supports both SELinux and AppArmor as source and target systems.
- Attempts to extract access violations from logs and generate approximate policies in the target system.
- Handles package management via `apt`, `dnf`, or `yum` as appropriate.
- Safe configuration updates with no forced overwrites.
- Requires root privileges and confirms actions before proceeding.

## Supported Distributions

- **Debian 10+**
- **Ubuntu 18.04+**
- **RHEL 8/9**
- **CentOS Stream 8/9**
- **Rocky Linux / AlmaLinux 8/9**
- **Fedora 36+**

## Installation

1. Copy the script to system binaries:

   ```bash
   sudo cp switch-sa /usr/bin/switch-sa
   ```

2. Make it executable:

   ```bash
   sudo chmod +x /usr/bin/switch-sa
   ```

## Usage

Run the tool with root privileges:

```bash
sudo switch-sa
```

The tool will:
- Detect the current MAC status (SELinux and AppArmor).
- Prompt for confirmation before switching.
- Disable the active system and enable the other.
- Attempt to generate basic policy rules from recent access denials.
- Suggest a reboot (required for full enforcement).

## Policy Conversion (Cross-Migration)

While full automatic policy translation is not possible due to fundamental differences between SELinux (type enforcement) and AppArmor (path-based access control), the tool performs best-effort rule extraction:

### SELinux → AppArmor
- Parses `audit.log` for `AVC denied` messages.
- Extracts process name, file path, and requested access type.
- Generates a basic AppArmor profile in `/etc/apparmor.d/usr.bin.<comm>`.

### AppArmor → SELinux
- Scans system logs for `apparmor="DENIED"` entries.
- Creates a template `.te` policy module in `/tmp/` to guide manual SELinux policy creation.

> **Note**: These are initial approximations. Manual review and refinement using `audit2allow`, `sealert`, or `aa-logprof` are strongly recommended.

## Requirements

- Python 3.6+
- Root privileges (`sudo`)
- `auditd` service (for SELinux logging)
- `systemd` (for service control)

## Limitations

- Do not run if both SELinux and AppArmor are active — this may cause conflicts.
- The tool does not support hybrid configurations.
- Policy conversion is heuristic and incomplete; it serves as a starting point.
- On SELinux enablement, a reboot with relabeling (`/.autorelabel`) is triggered.

## Safety

- Configuration files are modified in place with minimal changes.
- No data is deleted.
- Always back up `/etc/selinux/` and `/etc/apparmor.d/` before switching.

## License

This tool is provided as-is, without warranty. Use at your own risk.

## Author

Maintained by Mikhail Gubin for operational use and migration support.
