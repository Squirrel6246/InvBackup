# Inventory Backup (Fabric, server-side only)

For Minecraft **26.2** and **26.3** (Mojang names, unobfuscated, Java 25). Players do NOT need the mod;
the server needs Fabric Loader >= 0.19.5 and Fabric API. Commands need op (gamemaster) permission.

## Building
Requires **JDK 25**.

    ./gradlew build                                                          # 26.2 (default)
    ./gradlew build -Pminecraft_version=26.3 -Pfabric_api_version=0.162.0+26.3   # 26.3

Jars land in `build/libs/` as `invbackup-<version>+mc<mcversion>.jar` (use the one without `-sources`).
Each jar only loads on the Minecraft version it was built for. Check https://fabricmc.net/develop for newer
Fabric API versions.

Backups live in `<world>/invbackups/<uuid>/`. Config: `config/invbackup.json`.

## Commands (`/invbackup ...`)
| Command | What it does |
|---|---|
| `create <players>` | Manual backup of online player(s), e.g. `@a` |
| `list <player> [page]` | List backups (works for offline players too) |
| `view <player> <id>` | Show contents with click-to-take buttons |
| `open <player> <id>` | Open the backup as a chest GUI |
| `restore <player> <id> [target]` | Replace inventory with the backup (a `prerestore` backup is made first) |
| `take <player> <id> all` | Give yourself everything in the backup |
| `take <player> <id> <slot>` | Give yourself one stack (slot numbers are shown in `view`) |
| `delete <player> <id>` | Delete one backup |
| `prune` | Run cleanup now |
| `config` | Show settings |
| `config deaths\|join\|leave <true/false>` | Toggle automatic backups per event |
| `config maxAge <n> <unit>` | Delete backups older than n seconds/minutes/hours/days/weeks (e.g. `maxAge 12 hours`) |
| `config maxAge off` | Never delete by age |
| `config maxBackups <n>` | Keep at most n per player (0 = unlimited) |
| `config exemptManual <bool>` | Never prune manual backups |
| `config cleanupMinutes <n>` | Background cleanup interval |
| `config reload` | Re-read the JSON file |
