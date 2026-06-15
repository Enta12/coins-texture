# Coins — resource pack

Portraits for Yasuu19, DwayneThePhoque & FroggEater on items `coins:coin_yasuu19_gold`, `coins:coin_dwaynethephoque_gold`, and `coins:coin_froggeater_iron`.

**Minecraft 26.1** (resource pack format **84**) — also requires the datapack or plugin that registers these items.

## Installation

1. Download **`coins-texture.zip`** from the [GitHub releases](https://github.com/Enta12/coins-texture/releases) (not “Source code (zip)”).
2. Place the file in `%appdata%\.minecraft\resourcepacks\` **without extracting it**.
3. Enable the pack in *Options → Resource Packs*.

Alternative: use the pack folder as-is in `resourcepacks/` (root must contain `pack.mcmeta`).

## Adding a new coin

Each portrait coin needs a matching entry in **both** the datapack (game logic) and this resource pack (texture & model). The `item_model` ID must be identical on both sides.

### 1. Choose an item ID

Follow this naming pattern:

```
coin_{portrait}_{type}
```

| Part | Rule | Examples |
|------|------|----------|
| `portrait` | Lowercase, no spaces | `yasuu19`, `dwaynethephoque`, `froggeater` |
| `type` | Coin metal | `gold`, `iron` |

Full ID examples: `coin_yasuu19_gold`, `coin_froggeater_iron`  
In-game reference: `coins:coin_yasuu19_gold`

### 2. Prepare the texture

- Format: **PNG**, **16×16** pixels (standard Minecraft item size).
- Save as:

```
assets/coins/textures/item/coin_{portrait}_{type}.png
```

Example: `assets/coins/textures/item/coin_froggeater_iron.png`

> The atlas (`assets/coins/atlases/items.json`) already loads every file in `textures/item/` — no change needed there.

### 3. Create the model file

Create `assets/coins/models/item/coin_{portrait}_{type}.json`:

```json
{
	"parent": "minecraft:item/generated",
	"textures": {
		"layer0": "coins:item/coin_{portrait}_{type}"
	}
}
```

Example for FroggEater iron:

```json
{
	"parent": "minecraft:item/generated",
	"textures": {
		"layer0": "coins:item/coin_froggeater_iron"
	}
}
```

### 4. Create the item definition

Create `assets/coins/items/coin_{portrait}_{type}.json`:

```json
{
	"model": {
		"type": "minecraft:model",
		"model": "coins:item/coin_{portrait}_{type}"
	}
}
```

This file registers the custom item model used by the datapack via `minecraft:item_model`.

### 5. Update the datapack (required)

In the coins datapack, add a `give` rule in `data/coins/function/give.mcfunction` that references the same model ID:

```
minecraft:item_model="coins:coin_{portrait}_{type}"
```

Also update `help.mcfunction` and the datapack README with the new portrait and type.

Test in-game:

```
/function coins:give {count:1,portrait:"FroggEater",type:"iron"}
```

Without the resource pack, the item keeps the vanilla nugget appearance but still shows the custom name and lore.

### 6. Checklist

| Step | File / action |
|------|----------------|
| Texture | `assets/coins/textures/item/coin_….png` |
| Model | `assets/coins/models/item/coin_….json` |
| Item | `assets/coins/items/coin_….json` |
| Datapack | `give.mcfunction` + `help.mcfunction` |
| Optional | Update `pack.mcmeta` description |
| Release | Rebuild `coins-texture.zip` (see below) |

## Local folder vs `.zip` archive

The pack works as a **folder** but may fail as a **`.zip`** if the archive was created with default Windows tools (File Explorer, “Send to → Compressed folder”, `Compress-Archive`).

### Cause

These tools put **backslashes** (`\`) in zip paths:

```
assets\coins\textures\item\coin_yasuu19_gold.png   ❌
```

Minecraft Java expects **forward slashes** (`/`) in archives:

```
assets/coins/textures/item/coin_yasuu19_gold.png   ✅
```

| Mode | Behavior |
|------|----------|
| Folder in `resourcepacks/` | Windows handles paths → OK |
| Zip with `\` in entries | Game cannot resolve paths → broken pack or missing textures |

The GitHub “Source code” zip also includes a subfolder (`coins-texture-26.1/`): unusable as a Minecraft zip as-is. Use the **`coins-texture.zip`** release asset instead.

## Creating the archive for a release

From the pack root (where `pack.mcmeta` lives):

```powershell
$root = (Get-Location).Path
$outZip = Join-Path $root "coins-texture.zip"
if (Test-Path $outZip) { Remove-Item $outZip }

Add-Type -AssemblyName System.IO.Compression
Add-Type -AssemblyName System.IO.Compression.FileSystem

$zip = [System.IO.Compression.ZipFile]::Open($outZip, [System.IO.Compression.ZipArchiveMode]::Create)

Get-ChildItem -Recurse $root -File | Where-Object {
  $_.FullName -notmatch '\\.git\\' -and
  $_.Name -ne 'coins-texture.zip' -and
  $_.Name -ne '.gitignore' -and
  $_.Name -ne 'README.md'
} | ForEach-Object {
  $entry = $_.FullName.Substring($root.Length + 1).Replace('\', '/')
  [void][System.IO.Compression.ZipFileExtensions]::CreateEntryFromFile(
    $zip, $_.FullName, $entry, [System.IO.Compression.CompressionLevel]::Optimal
  )
}

$zip.Dispose()
Write-Host "Created: $outZip"
```

Verify that zip entries use `/` (not `\`).

With **7-Zip**: zip the **contents** of the folder (`pack.mcmeta`, `assets/`), not the parent folder.

Then publish `coins-texture.zip` as a release asset on GitHub.
