# RVS In A Box

A complete Raven Shield dedicated server, web dashboard, and fast download server - deployed with a single command. No Windows, no Wine config, no manual file set up.
    
    Disclaimer: You MUST own a legitimate copy of the game to use this.

## What You Get

- **Game Server** - Raven Shield 1.60 running via Wine with OpenRVS, N4Admin, URLPost, and N4IDMod
- **Dashboard** - Live server status, player statistics, round history, and remote administration via [RVSDash](https://github.com/ericreinsmidt/RVSDash)
- **Fast Downloads** - Caddy serves custom maps and mod files directly from the custom/ directory for fast client downloads

## Prerequisites

- An x86_64 Linux host with [Docker](https://docs.docker.com/engine/install/) installed (ARM is not supported)
- Minimum 512 MB RAM, 1 CPU core (full stack uses ~220 MB RAM at idle)
- ~6.5 GB of disk space (grows with custom maps and stats data)
- A static public IP or domain name
- The following ports forwarded to your host:

| Port | Protocol | Purpose |
|------|----------|---------|
| 7777 | UDP | Game traffic |
| 8777 | UDP | Server beacon (N4Admin) |
| 9777 | UDP | Client beacon port |
| 80 | TCP | Fast downloads + redirect server |
| 2003 | TCP | Dashboard (configurable) |

## Quick Start

    git clone https://github.com/ericreinsmidt/rvs-in-a-box.git
    cd rvs-in-a-box
    python3 setup.py
    docker compose pull
    docker compose up -d

The setup script will walk you through server configuration and generate all necessary files.

## Configuration

Running `setup.py` generates three files and two directories:

| File / Directory | Purpose |
|------------------|---------|
| `.env` | All server settings - edit this to reconfigure |
| `docker-compose.yml` | Container definitions - no need to edit |
| `Caddyfile` | Redirect server config - no need to edit |
| `custom/` | Drop custom maps and archives here |
| `data/` | RVSDash stats database and audit log (persists across restarts) |

### Settings Reference

#### Server Identity

| Variable | Default | Description |
|----------|---------|-------------|
| `SERVER_NAME` | Raven Shield Server | Server name shown in the game lobby |
| `MOTD` | Welcome! Have fun and play fair. | Message of the day |
| `SERVER_IDENT` | rvs1 | Unique identifier for stats tracking |

#### Passwords

| Variable | Default | Description |
|----------|---------|-------------|
| `ADMIN_PASSWORD` | (required) | In-game admin password |
| `N4ADMIN_PASSWORD` | (required) | Dashboard and N4Admin query password |

#### Game Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `GAME_MODE` | R6Game.R6TerroristHuntCoopGame | Game mode class |
| `MAX_PLAYERS` | 8 | Maximum player count |
| `NB_TERRO` | 30 | Terrorist count |
| `ROUND_TIME` | 300 | Round time in seconds |
| `BETWEEN_ROUND_TIME` | 45 | Time between rounds in seconds |
| `BOMB_TIME` | 300 | Bomb timer in seconds |
| `ROUNDS_PER_MATCH` | 10 | Rounds per match |
| `DIFF_LEVEL` | 2 | Difficulty (1=Easy, 2=Normal, 3=Hard) |
| `FRIENDLY_FIRE` | True | Enable friendly fire |
| `FORCE_FIRST_PERSON` | True | Force first person weapon view |
| `ROTATE_MAP` | True | Enable map rotation |
| `MAPS` | Training,Airport,Bank,... | Comma-separated map list |

#### Network

| Variable | Default | Description |
|----------|---------|-------------|
| `PUBLIC_IP` | (required) | Your public IP or domain name |
| `SERVER_BEACON_PORT` | 8777 | Beacon port (game port + 1000) |
| `RVSDASH_PORT` | 2003 | Dashboard web port |

## Available Maps

Airport, Alpines, Bank, CityStreet_LG, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse

## Game Modes

| Mode | Class |
|------|-------|
| Terrorist Hunt Co-op | R6Game.R6TerroristHuntCoopGame |
| Hostage Rescue Co-op | R6Game.R6HostageRescueCoopGame |
| Hostage Rescue Advanced | R6Game.R6HostageRescueAdvGame |
| Team Deathmatch | R6Game.R6TeamDeathMatchGame |
| Bomb | R6Game.R6TeamBomb |
| Escort Pilot | R6Game.R6EscortPilotGame |
| Deathmatch | R6Game.R6DeathMatch |
| Lone Wolf | R6Game.R6LoneWolfGame |

## Optional Mods

RVS In A Box includes three optional server-side mods that can be enabled during setup:

| Mod | Description |
|-----|-------------|
| 200 Ammo | Doubles ammo capacity for all weapons |
| Run N Gun | Removes the speed penalty when firing weapons |
| Santa Hat | Holiday cosmetic that adds santa hats to players |

Mods are toggled during setup.py. To change them later, edit the MOD_ variables in .env and restart:

    docker compose down
    docker compose up -d

## Custom Maps

To add custom maps, drop files into the custom/ directory (created by setup.py). You can drop in .7z or .zip archives directly — they are automatically extracted at startup. Or drop in loose files of any type, all mixed together. The entrypoint sorts everything into the correct game directories.

    cp /path/to/MyMapPack.7z custom/
    docker compose down
    docker compose up -d

Or with loose files:

    cp /path/to/MyMap.rsm /path/to/MyMap_T.utx custom/
    docker compose down
    docker compose up -d

Supported file types:

| Extension | Destination | Description |
|-----------|-------------|-------------|
|`.rsm` | maps/ | Map files |
|`.ini` | maps/ | Map metadata |
|`.utx` | textures/ | Texture packages |
|`.usx` | staticmeshes/ | Static mesh packages |
|`.u` | System/ | Code/script packages |
|`.int` | System/ | Localization files |
|`.ukx` | animations/ | Animation packages |
|`.uax` | sounds/ | Sound packages |
|`.sb0` | sounds/ | Sound bank files |
|`.tph` | template/ | Hostage placement templates |
|`.tpt` | template/ | Terrorist placement templates |

The custom/ directory is served directly by Caddy for fast HTTP downloads. Players connecting to your server will automatically download custom files from here. Enabled mod files are also copied into custom/ at startup so players can download them.

Remember to add custom maps to your MAPS list in `.env`.

## Data and Backups

RVSDash stores player stats, round history, and an audit log in the data/ directory:

| File | Description |
|------|-------------|
| `data/rvsstats.sqlite3` | SQLite database with all player stats and round history |
| `data/ingest.ndjson` | Newline-delimited JSON audit log of every round received |

To back up your stats, copy the data/ directory:

    cp -r data/ /path/to/backup/

The data/ directory persists across container restarts and image updates. It is only removed if you manually delete it or the rvs-in-a-box/ directory.

## Managing Your Server

### View logs

    docker compose logs -f

### Stop the server

    docker compose down

### Reconfigure

**Quick changes** — edit `.env` directly, then restart:

    docker compose down
    docker compose up -d

**Full reconfigure** — re-run the setup wizard (overwrites `.env`, `docker-compose.yml`, and `Caddyfile`):

    python3 setup.py
    docker compose down
    docker compose up -d

### Update to latest images

    docker compose pull
    docker compose down
    docker compose up -d

## Troubleshooting

### Server starts but nobody can join

- Verify ports `7777/udp`, `8777/udp`, and `9777/udp` are forwarded to your host
- Check your firewall allows inbound UDP on those ports
- Confirm `PUBLIC_IP` in `.env` matches your actual public IP

### Dashboard shows No UDP response

- The game server may still be starting - wait 30 seconds and refresh
- Verify `SERVER_BEACON_PORT` in `.env` matches the beacon port (default 8777)

### Players get Downloading but it hangs

- Verify port `80/tcp` is forwarded to your host
- Check that the redirect container is running: `docker compose ps`

### Server not appearing in OpenRVS lobby

- The server may need to be manually registered at openrvs.org

## Security

The RVSDash admin page (`http://your-ip:2003/admin`) has **no built-in authentication**. Anyone who can reach the dashboard port can kick players, ban users, change maps, and restart the server.

The status and stats pages are read-only and safe to expose publicly.

If your server is internet-facing, restrict access to the admin page using one of:

- [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/applications/) — identity-aware access control
- Reverse proxy with IP allowlisting (nginx/Caddy)
- VPN — only expose the dashboard on a private network
- Firewall rules — restrict port 2003 to trusted IPs

## Credits

- [OpenRVS](https://github.com/OpenRVS-devs/OpenRVS) - .Twi, Will, Tony, and the OpenRVS community
- N4Admin and URLPost - Neil Popplewell (Neo4E656F), copyright 2003-2004
- [N4IDMod](https://dateranoth.com) - Dateranoth and .Twi, copyright 2020
- [OpenRenderFix](https://github.com/OpenRVS-devs/OpenRVS) - OpenRVS team
- [RVSDash](https://github.com/ericreinsmidt/RVSDash) - Eric Reinsmidt

## License

MIT
