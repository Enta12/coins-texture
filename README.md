# Coins — resource pack

Portraits Yasuu19 & DwayneThePhoque pour les items `coins:coin_yasuu19_gold` et `coins:coin_dwaynethephoque_gold`.

**Minecraft 26.1** (format resource pack **84**) — nécessite aussi le datapack ou plugin qui enregistre ces items.

## Installation

1. Télécharger **`coins-texture.zip`** depuis les [releases GitHub](https://github.com/Enta12/coins-texture/releases) (pas « Source code (zip) »).
2. Placer le fichier dans `%appdata%\.minecraft\resourcepacks\` **sans l’extraire**.
3. Activer le pack dans *Options → Packs de ressources*.

Alternative : utiliser le dossier du pack tel quel dans `resourcepacks/` (racine = `pack.mcmeta`).

## Dossier local vs archive `.zip`

Le pack fonctionne en **dossier** mais peut échouer en **`.zip`** si l’archive a été créée avec les outils Windows par défaut (Explorateur, « Envoyer vers → Dossier compressé », `Compress-Archive`).

### Cause

Ces outils mettent des **antislashs** (`\`) dans les chemins du zip :

```
assets\coins\textures\item\coin_yasuu19_gold.png   ❌
```

Minecraft Java attend des **slashs** (`/`) dans les archives :

```
assets/coins/textures/item/coin_yasuu19_gold.png   ✅
```

| Mode | Comportement |
|------|----------------|
| Dossier dans `resourcepacks/` | Windows gère les chemins → OK |
| Zip avec `\` dans les entrées | Le jeu ne résout pas les chemins → pack cassé ou textures manquantes |

Le zip « Source code » des releases GitHub a en plus un sous-dossier (`coins-texture-26.1/`) : inutilisable tel quel comme zip Minecraft. Utiliser l’asset **`coins-texture.zip`**.

## Créer l’archive pour une release

Depuis la racine du pack (là où se trouve `pack.mcmeta`) :

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
Write-Host "Cree : $outZip"
```

Vérifier que les entrées du zip utilisent bien `/` (pas `\`).

Avec **7-Zip** : zipper le **contenu** du dossier (`pack.mcmeta`, `assets/`), pas le dossier parent.

Puis publier `coins-texture.zip` en asset de release sur GitHub.
