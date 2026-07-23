# Palworld Dedicated Server — Docker
 
Run a [Palworld](https://www.pocketpair.jp/palworld) dedicated server in a container with a single `docker compose up`.
 
Built on top of the official [pocketpairjp/palworld-dedicated-server-docker](https://github.com/pocketpairjp/palworld-dedicated-server-docker) image.
 
---
 
## Table of contents
 
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Backups](#backups)
- [Troubleshooting](#troubleshooting)
- [References](#references)
---
 
## Requirements
 
Official hardware requirements: <https://docs.palworldgame.com/getting-started/requirements/>
 
| Parameter | Requirement |
|---|---|
| **CPU** | 4 cores or more (recommended) |
| **Memory** | 16 GB. 32 GB or more recommended for larger worlds. 8 GB is bootable, but increases the risk of the server crashing from out-of-memory. |
| **Network** | UDP port `8211` (default, changeable). Port forwarding must be configured on the router. |
| **Storage** | Fast SSD recommended. Low-performance storage may corrupt saved data. |
| **OS** | Linux 64-bit (Debian, Ubuntu, AlmaLinux, etc.) / Windows 64-bit |
 
You also need **Docker Engine** and the **Compose plugin** installed: <https://docs.docker.com/engine/install/>
 
Check your installation:
 
```bash
docker --version
docker compose version
```
 
### Ports
 
| Port | Protocol | Purpose | Required |
|---|---|---|---|
| `8211` | UDP | Game traffic | Yes |
| `25575` | TCP | RCON (remote administration) | Optional |
 
> [!WARNING]
> Only forward the RCON port on your router if you really need remote administration, and always set a strong `AdminPassword`.
 
---
 
## Installation
 
### 1. Clone the repository
 
```bash
git clone https://github.com/avl12ng/palworld_svr_docker.git
cd palworld_svr_docker
```
 
### 2. Check that the Compose file is present
 
```bash
ls
```
 
The output must contain `compose.yaml`.
 
---
 
## Configuration
 
Server settings live in:
 
```
./Saved/Config/LinuxServer/PalWorldSettings.ini
```
 
Full list of available options: <https://docs.palworldgame.com/settings-and-operation/configuration/>
 
Minimal example:
 
```ini
[/Script/Pal.PalGameWorldSettings]
OptionSettings=(ServerPassword="YourServerPassword",AdminPassword="AdminPassword",DeathPenalty=Item,bAllowClientMod=True,bIsUseBackupSaveData=True,RCONEnabled=True,RCONPort="25575",ServerName="YourServerName",ServerDescription="YourServerDescription")
```
 
> [!IMPORTANT]
> `OptionSettings` must stay on a **single line**, with no spaces around the `=` signs and no line breaks between parameters. A malformed line makes the server silently fall back to default settings.
 
Restart the container after any change for it to take effect:
 
```bash
docker compose restart
```
 
---
 
## Usage
 
All commands must be run from the folder containing `compose.yaml`.
 
### Start the server
 
```bash
docker compose up -d
```
 
First start downloads the server files and can take several minutes.
 
### Stop the server
 
```bash
docker compose down
```
 
### Restart the server
 
```bash
docker compose restart
```
 
### View logs
 
```bash
docker compose logs          # full log
docker compose logs -f       # follow live
docker compose logs --tail 100
```
 
### Update the server
 
```bash
docker compose pull
docker compose up -d
```
 
---
 
## Backups
 
World data is stored under `./Saved/`. Stop the server before copying it to guarantee a consistent snapshot:
 
```bash
docker compose down
tar czf palworld-backup-$(date +%F).tar.gz ./Saved
docker compose up -d
```
 
Keeping `bIsUseBackupSaveData=True` in the configuration is also strongly recommended.
 
---
 
## Troubleshooting
 
| Symptom | Things to check |
|---|---|
| Players cannot connect | UDP `8211` forwarded on the router; server actually listening (`docker compose logs`); public IP not behind CGNAT |
| Settings are ignored | `OptionSettings` on one line, correct file path, container restarted after the edit |
| Container restarts in a loop | Available RAM, free disk space, `docker compose logs` output |
| Corrupted save | Restore from `./Saved` backup; check that storage is an SSD |
 
---
 
## References
 
- Official server documentation — <https://docs.palworldgame.com/>
- Configuration reference — <https://docs.palworldgame.com/settings-and-operation/configuration/>
- Upstream image — <https://github.com/pocketpairjp/palworld-dedicated-server-docker>
