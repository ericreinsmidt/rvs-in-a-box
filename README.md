# RVS In A Box

A complete Raven Shield dedicated server, web dashboard, and fast download server — deployed with a single command. No compiling, no game files, no Windows required.
    
    Disclaimer: You MUST own a legitimate copy of the game to use this.

## What You Get

- **Game Server** — Raven Shield 1.60 running via Wine with OpenRVS, N4Admin, URLPost, and N4IDMod
- **Dashboard** — Live server status, player statistics, round history, and remote administration via [RVSDash](https://github.com/ericreinsmidt/RVSDash)
- **Fast Downloads** — Caddy-powered HTTP redirect server for custom maps and textures

## Prerequisites

- A Linux host with [Docker](https://docs.docker.com/engine/install/) installed
- A static public IP or domain name
- The following ports forwarded to your host:

| Port | Protocol | Purpose |
|------|----------|---------|
| 7777 | UDP | Game traffic |
| 8777 | UDP | Server beacon (N4Admin) |
| 9777 | UDP | Beacon response |
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

Running setup.py generates three files:

| File | Purpose |
|------|---------|
| .env | All server settings — edit this to reconfigure |
| docker-compose.yml | Container definitions — no need to edit |
| Caddyfile | Redirect server config — no need to edit |

### Settings Reference

#### Server Identity

| Variable | Default | Description |
|----------|---------|-------------|
| SERVER_NAME | Raven Shield Server | Server name shown in the game lobby |
| MOTD | Welcome! Have fun and play fair. | Message of the day |
| SERVER_IDENT | rvs1 | Unique identifier for stats tracking |

#### Passwords

| Variable | Default | Description |
|----------|---------|-------------|
| ADMIN_PASSWORD | (required) | In-game admin password |
| N4ADMIN_PASSWORD | (required) | Dashboard and N4Admin query password |

#### Game Settings

| Variable | Default | Description |
|----------|---------|-------------|
| GAME_MODE | R6Game.R6TerroristHuntCoopGame | Game mode class |
| MAX_PLAYERS | 8 | Maximum player count |
| NB_TERRO | 30 | Terrorist count |
| ROUND_TIME | 300 | Round time in seconds |
| BETWEEN_ROUND_TIME | 45 | Time between rounds in seconds |
| BOMB_TIME | 300 | Bomb timer in seconds |
| ROUNDS_PER_MATCH | 10 | Rounds per match |
| DIFF_LEVEL | 2 | Difficulty (1=Easy, 2=Normal, 3=Hard) |
| FRIENDLY_FIRE | True | Enable friendly fire |
| FORCE_FIRST_PERSON | True | Force first person weapon view |
| ROTATE_MAP | True | Enable map rotation |
| MAPS | Training,Airport,Bank,... | Comma-separated map list |

#### Network

| Variable | Default | Description |
|----------|---------|-------------|
| PUBLIC_IP | (required) | Your public IP or domain name |
| SERVER_BEACON_PORT | 8777 | Beacon port (game port + 1000) |
| RVSDASH_PORT | 2003 | Dashboard web port |

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

## Managing Your Server

### View logs

    docker compose logs -f

### Stop the server

    docker compose down

### Reconfigure

    python3 setup.py
    docker compose down
    docker compose up -d

### Update to latest images

    docker compose pull
    docker compose down
    docker compose up -d

## Troubleshooting

### Server starts but nobody can join

- Verify ports 7777/udp, 8777/udp, and 9777/udp are forwarded to your host
- Check your firewall allows inbound UDP on those ports
- Confirm PUBLIC_IP in.env matches your actual public IP

### Dashboard shows No UDP response

- The game server may still be starting — wait 30 seconds and refresh
- Verify `SERVER_BEACON_PORT` in `.env` matches the beacon port (default 8777)

### Players get Downloading but it hangs

- Verify port 80/tcp is forwarded to your host
- Check that the redirect container is running: docker compose ps

### Server not appearing in OpenRVS lobby

- The server may need to be manually registered at openrvs.org

## Credits

- [OpenRVS](https://github.com/OpenRVS-devs/OpenRVS) — Twi, Will, Tony, and the OpenRVS community
- N4Admin and URLPost — Neil Popplewell (Neo4E656F), copyright 2003-2004
- [N4IDMod](https://dateranoth.com) — Dateranoth and.Twi, copyright 2020
- [OpenRenderFix](https://github.com/OpenRVS-devs/OpenRVS) — OpenRVS team
- [RVSDash](https://github.com/ericreinsmidt/RVSDash) — Eric Reinsmidt

## License

MIT
