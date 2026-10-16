# Inventory Backup (Fabric, server-side only)

For Minecraft **26.2** and **26.3** (Mojang names, unobfuscated, Java 25). Players do NOT need the mod;
the server needs Fabric Loader >= 0.19.5 and Fabric API. Commands need op (gamemaster) permission.

## Building

Requires **JDK 25**.

&#x20;   ./gradlew build                                                          # 26.2 (default)
./gradlew build -Pminecraft\_version=26.3 -Pfabric\_api\_version=0.162.0+26.3   # 26.3



Jars land in `build/libs/` as `invbackup-<version>+mc<mcversion>.jar` (use the one without `-sources`).
Each jar only loads on the Minecraft version it was built for. Check https://fabricmc.net/develop for newer
Fabric API versions.

Backups live in `<world>/invbackups/<uuid>/`. Config: `config/invbackup.json`.

## Backup references

Every player has their own backup numbers: **#1, #2, #3 ...** Numbers are never reused, even after old backups are pruned.
Wherever a command takes `<backup>` you can use:

|Reference|Meaning|
|-|-|
|`14` (or `"#14"`, quotes required for the `#`)|backup #14|
|`latest` / `oldest`|newest / oldest backup|
|`latest-death`, `latest-join`, `latest-leave`, `latest-manual`, `latest-prerestore`|newest backup of that kind|
|`before-raid`|a label you gave it (case-insensitive)|

Labels: 1-32 characters, letters/numbers and `- . +`; can't be only digits or start with `latest`; unique per player.
Chat buttons in `list`/`view` fill in the right reference for you.

## Commands (`/invbackup ...`)

|Command|What it does|
|-|-|
|`create <players> \\\[label]`|Manual backup of online player(s), e.g. `@a`, optionally labeled|
|`list <player> \\\[page]`|Every backup: #number, label, date/age, reason, stack/item counts, dimension and coordinates (works for offline players too)|
|`view <player> <backup>`|Show contents with click-to-take buttons|
|`open <player> <backup>`|Open the backup as a chest GUI|
|`label <player> <backup> <label>`|Name a backup|
|`unlabel <player> <backup>`|Remove the name|
|`restore <player> <backup> \\\[target]`|Replace inventory with the backup (a `prerestore` backup is made first)|
|`take <player> <backup> all`|Give yourself everything in the backup|
|`take <player> <backup> <slot>`|Give yourself one stack (slot numbers are shown in `view`)|
|`delete <player> <backup>`|Delete one backup|
|`prune`|Run cleanup now|
|`config`|Show settings|
|`config deaths\|join\|leave <true/false>`|Toggle automatic backups per event|
|`config maxAge <n> <unit>` / `maxAge off`|Delete backups older than n seconds/minutes/hours/days/weeks|
|`config maxBackups <n>`|Keep at most n per player (0 = unlimited)|
|`config exemptManual <bool>`|Never prune manual backups|
|`config exemptLabeled <bool>`|Never prune labeled backups|
|`config cleanupMinutes <n>`|Background cleanup interval|
|`config reload`|Re-read the JSON file|

Backups made by the previous version (named by timestamp only) are numbered automatically, oldest first, the first time they're listed.

## Storage usage (estimates)

Each backup is one gzip-compressed file. These figures are rough estimates, not measurements; check the
`<world>/invbackups` folder after a few days of normal play to see real numbers.

**Size of one backup**

|Inventory|Approx. size|
|-|-|
|Typical survival inventory|1-3 KB|
|Full inventory of enchanted, renamed or lore-heavy gear|5-15 KB|
|Inventory holding filled shulker boxes or bundles|20-100+ KB|

Items inside containers are stored too, so filled shulker boxes are the biggest cost. Small files also take at
least one disk block (usually 4 KB) each, so a typical backup counts as about 4 KB on disk.

**Worst case with the default limits (50 backups per player, 7 days)**

|Player type|Per player|50 players|200 players|
|-|-|-|-|
|Typical|\~200 KB|\~10 MB|\~40 MB|
|Gear-heavy|\~500 KB|\~25 MB|\~100 MB|
|Shulker hoarder|\~2-5 MB|\~100-250 MB|\~400 MB-1 GB|

Most players are in the first two rows, so realistic totals are tens of megabytes.

**What makes it grow**

* `maxBackups 0` together with `maxAge off` removes all limits, so storage grows forever. Keep at least one set.
* Backups exempted from pruning (`exemptManual`, `exemptLabeled`) are never deleted.
* Join and leave backups make 2 per play session, so busy servers hit the per-player cap quickly. The cap, more than
the age limit, is usually what controls storage, and routine join/leave backups can push old death backups out.
If death backups matter most, turn off join or leave backups, or label the ones you want to keep and enable
`exemptLabeled`.
* Automatic backups skip empty inventories.

