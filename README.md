# netFIELD Linux Client

## netFIELD support for Linux devices

The netFIELD Linux Client (NFLC) installs and manages netFIELD services and
Cockpit extensions on supported Debian-family Linux devices. It detects the
host operating system, resolves component dependencies, downloads
version-pinned packages, applies changes through APT, and verifies the final
package and service state.

NFLC also provides guided workflows for onboarding a device to a netFIELD
instance and offboarding it again.

## Supported systems

NFLC supports:

| Operating system | Architectures |
|---|---|
| Ubuntu 22.04 | `amd64`, `arm64` |
| Ubuntu 24.04 | `amd64`, `arm64` |
| Debian 12 | `amd64`, `arm64` |
| Debian 13 | `amd64`, `arm64` |
| Raspberry Pi OS (Raspbian) 11 | `arm64` |
| Raspberry Pi OS (Raspbian) 12 | `arm64` |

Package availability can differ by component. The Cockpit extensions, App
Dashboard, and runtime packages cover the applicable systems in this matrix.
The automatic download of the core Cockpit package set is currently configured
for Ubuntu 22.04 and 24.04 on `amd64`; other targets require a matching local
Cockpit package set. NFLC stops with an error when a required package is not
available for the detected operating-system version and architecture.

Check the host identity before installation:

```bash
cat /etc/os-release
dpkg --print-architecture
```

## System requirements

The target must provide:

- Bash 5.1 or newer
- APT and Debian package tools (`apt-get`, `dpkg`, and `dpkg-query`)
- systemd and `systemctl`
- `curl`, `jq`, `tar`, and `sha256sum`
- `ss` from the `iproute2` package
- Internet access to the package sources declared by this release
- `root` access or `sudo` for installation, upgrade, removal, onboarding, and
  offboarding

Use a current package index before starting:

```bash
sudo apt-get update
```

Existing package versions can prevent APT from resolving an installation.
Review the dry-run plan and APT output before changing production devices.

## Start the client

Extract the downloaded release and enter its directory:

```bash
tar -xzf netfield-linux-client_<version>.tar.gz
cd netfield-linux-client-<version>
```

Confirm the client version and inspect the device without making changes:

```bash
./bin/nflc version
./bin/nflc status
```

The client runs directly from the extracted directory. Keep `bin/`, `lib/`,
`manifests/`, and `nflc.config` together.

## Install components

Start with an interactive dry run. A dry run resolves and validates the plan
without downloading or installing packages:

```bash
./bin/nflc install --interactive --dry-run
```

Apply an interactive installation with administrative privileges:

```bash
sudo ./bin/nflc install --interactive
```

Components can also be selected explicitly:

```bash
./bin/nflc install cockpit certificate --dry-run
sudo ./bin/nflc install cockpit certificate --yes
```

NFLC installs declared dependencies automatically. It first checks matching
local packages, then its verified cache. Component packages are downloaded from
manifest-pinned public GitHub releases. Runtime prerequisites are downloaded
from their official upstream package sources or public GitHub releases.
Downloaded assets are accepted only when their SHA-256 checksum and Debian
package metadata match the manifest.

## Available components

| Component ID | Purpose |
|---|---|
| `appDashboard` | Manage applications and workloads on the device |
| `cockpit` | Provide the Cockpit web administration service |
| `certificate` | Manage device certificates and trust configuration |
| `docker` | Manage Docker containers through Cockpit |
| `generalSettings` | Configure general netFIELD device settings |
| `iotedge_docker` | Manage IoT Edge workloads running with Docker |
| `logDiagnostics` | Inspect system logs and diagnostic information |
| `networkservices` | Configure network interfaces and device services |
| `onboard` | Install the Cockpit onboarding extension |
| `swupdate` | Manage software updates |

The `onboard` component is different from the `nflc onboard` command. Installing
the component adds a Cockpit extension; running the command registers the
device with a cloud instance.

## Use Cockpit

After Cockpit has been installed, open this URL from a browser that can reach
the device:

```text
https://<device-address>:9090/
```

Cockpit uses HTTPS. A browser can display a certificate warning when the device
uses a locally generated or otherwise untrusted certificate.

Check the service locally with:

```bash
systemctl status cockpit.socket --no-pager
curl --insecure https://localhost:9090/
```

## Manage installed components

Inspect state:

```bash
./bin/nflc status
./bin/nflc status cockpit --output json
```

Preview and apply an upgrade:

```bash
./bin/nflc upgrade certificate --dry-run
sudo ./bin/nflc upgrade certificate --yes
```

Preview and apply removal:

```bash
./bin/nflc remove certificate --dry-run
sudo ./bin/nflc remove certificate --yes
```

Removal acts only on the components explicitly named in the command. Review the
plan before confirming it.

## Onboard and offboard a device

Run the guided onboarding workflow:

```bash
sudo ./bin/nflc onboard --interactive
```

Run the guided offboarding workflow before retiring, reimaging, or moving an
onboarded device to another instance:

```bash
sudo ./bin/nflc offboard --interactive
```

Onboarding and offboarding perform real changes and do not support `--dry-run`.
They can install or remove provider runtimes and communicate with the selected
cloud instance.

Do not manually delete `/var/lib/nflc/onboarding.json` or `/etc/netfield.io`
while the device is onboarded. These files connect the local device state with
its cloud registration and are required for a normal offboarding operation.

## Troubleshooting

When an operation fails:

1. Read the complete APT or NFLC error before retrying.
2. Refresh package indexes with `sudo apt-get update` when downloads return
   `404 Not Found`.
3. Run the command with `--dry-run --output json` to inspect the selected host,
   versions, dependencies, and actions.
4. Check package state with `dpkg-query -W '<package-name>'`.
5. Check a failed service with `systemctl status <unit> --no-pager` and
   `journalctl -u <unit> --no-pager`.

Do not bypass Debian dependencies with `dpkg --force-*`. Forced installation
can leave Cockpit extensions or runtime services unusable.

For product support, contact Hilscher at:

<https://www.hilscher.com/support/contact/>

General administration of Linux, Docker, Azure IoT Edge, and other third-party
software remains the responsibility of the device administrator and the
respective software vendor.

## Trademarks

`netFIELD` is a trademark of Hilscher Gesellschaft fuer Systemautomation mbH.
Other product names are trademarks of their respective owners.

Copyright Hilscher Gesellschaft fuer Systemautomation mbH. All rights
reserved.
