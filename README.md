# Voiceover Glass Reel

> **Skill d'agent IA** qui transforme **n'importe quelle voix off** (et un script optionnel) en **reel documentaire 16:9** monté nativement dans **After Effects** : coupe des pauses et respirations, storyboard, sourcing gratuit des images (Wikimedia Commons, Pixabay, logos thesvg), cadres de verre flottants, grille en perspective, gros mots, cartes photo, flashs, SFX synthétisés, rendu 1080p H.264.

[![Licence : MIT](https://img.shields.io/badge/licence-MIT-blue.svg)](LICENSE)
[![Plateforme](https://img.shields.io/badge/rendu-macOS%20%2B%20After%20Effects-lightgrey.svg)](#3-prérequis)
[![MCP](https://img.shields.io/badge/MCP-Pixabay%20%2B%20thesvg-orange.svg)](#5-installer-les-mcp-utilisés-par-le-skill)
[![Agent Skills](https://img.shields.io/badge/format-SKILL.md-purple.svg)](https://opencode.ai/v2/docs/skills)

---

## Sommaire

1. [Ce que fait le skill](#1-ce-que-fait-le-skill)
2. [Le look et le pipeline](#2-le-look-et-le-pipeline)
3. [Prérequis](#3-prérequis)
4. [Installer le skill](#4-installer-le-skill)
5. [Installer les MCP utilisés par le skill](#5-installer-les-mcp-utilisés-par-le-skill)
6. [Utilisation complète](#6-utilisation-complète)
7. [Structure du dépôt](#7-structure-du-dépôt)
8. [Sources gratuites, licences et attribution](#8-sources-gratuites-licences-et-attribution)
9. [Dépannage](#9-dépannage)
10. [FAQ](#10-faq)
11. [Licence](#11-licence)

---

## 1. Ce que fait le skill

Vous donnez **un fichier audio de voix off** (ou une vidéo dont on extrait l'audio), éventuellement le **script**. L'agent :

- convertit l'audio et **coupe toutes les pauses et respirations** ;
- transcrit avec les **temps par mot** (transcription on-device sur macOS 26) ;
- écrit une **histoire visuelle** : époque, lieu, personnages récurrents, décors, noir et blanc ou couleur ;
- source **gratuitement** les clips, portraits, archives et logos ;
- construit le montage **nativement dans After Effects** (compositions, textes, cadres de verre, grille) ;
- vérifie des images de contrôle, corrige, puis rend un **MP4 1080p H.264** ;
- laisse un projet AE propre (`reel.aep`) que vous pouvez retoucher à la main.

**Sortie :** `out/reel_1080.mp4` — 1920×1080, 25 fps, ~40 Mbps, avec son.

**Look par défaut :** documentaire archival noir et blanc années 1950–60, chaque clip dans un cadre de verre arrondi avec une fine bordure lumineuse et un léger push-in, grille en perspective derrière les listes avec les mots-clés qui changent de couleur au moment où ils sont dits, grandes captures, pages typewriter / mot géant / cartes photo, flashs et négatifs sur les coupes, SFX synthétisés.

> **Gratuit par défaut** : Pixabay, Wikimedia Commons, thesvg, ffmpeg Ken-Burns, SFX synthétisés. Aucun crédit, aucun budget à approuver. La génération IA payante (Higgsfield) n'existe qu'en appendice, sur demande explicite.

---

## 2. Le look et le pipeline

| Étape | Ce qui se passe | Commande |
|---|---|---|
| 1 | Dossier projet + audio coupé + transcription | `reel.py prep` |
| 2 | Lecture de l'index des mots | `reel.py words` |
| 3 | Écriture de `story.json` | l'agent |
| 4 | Sourcing des assets (clips, portraits, logos, musique) | MCP + API |
| 5 | Découpage des chunks d'audio par shot hôte | `reel.py chunks` |
| 6 | Alignement lip-sync | `reel.py align` |
| 7 | Fond vert de la TV | `reel.py screen` |
| 8 | SFX synthétisés | `reel.py sfx` |
| 9 | Plan + checks | `reel.py plan` |
| 10 | Build AE | `build.sh` |
| 11 | Rendu de contrôle puis final | `render.sh --half` / `render.sh` |

Types de shots disponibles dans `story.json` : `host` (présentateur dans la TV), `clip` (footage dans le cadre de verre), `type` (page machine à écrire), `giant` (mot géant + label rouge + sous-ligne serif), `cards` (cartes photo avec badges logo), `list` (grille en perspective). Effets : `invert`, `strobe`, `burn`.

---

## 3. Prérequis

### Système

| Élément | Requis | Notes |
|---|---|---|
| macOS | **26 ou plus** | nécessaire pour la transcription on-device (`SpeechAnalyzer`) |
| After Effects | **2024 ou plus**, ouvert | le skill pilote AE via scripting |
| Xcode Command Line Tools | oui | `xcode-select --install`, pour `swiftc` |
| Python | 3.9+ | bibliothèque standard uniquement |
| ffmpeg + curl | oui | déjà utilisés par le moteur |
| Node.js | 18+ | pour les serveurs MCP (`npx`) |
| Git | any | pour cloner le skill |

> **Windows** : le rendu n'est **pas** pris en charge (After Effects + transcription Apple + binaires Swift). Vous pouvez toutefois préparer la transcription et le sourcing sur PC, puis transférer le dossier projet sur le Mac pour `prep → build → render`. Voir la FAQ.

### Réglage After Effects indispensable

Dans After Effects :

```text
Settings → Scripting & Expressions → Allow Scripts to Write Files and Access Network
```

Sans cette option, `build.sh` et `render.sh` ne peuvent pas écrire le projet ni lancer le rendu.

### Autorisation macOS

Au premier lancement, macOS demande d'autoriser le terminal à contrôler After Effects :

```text
Réglages Système → Confidentialité et sécurité → Automatisation
```

La transcription peut aussi demander l'accès à la **Reconnaissance vocale** :

```text
Réglages Système → Confidentialité et sécurité → Reconnaissance vocale
```

### Outils compilés

```bash
cd ~/.config/opencode/skills/voiceover-glass-reel
chmod +x scripts/*.sh
./scripts/build_tools.sh
```

Cela compile `scripts/bin/{to_wav,transcribe,facebox,frames}` avec `swiftc`.

---

## 4. Installer le skill

Le dépôt **est** le dossier du skill : `SKILL.md` est à la racine.

### 4.1 OpenCode (recommandé)

```bash
mkdir -p ~/.config/opencode/skills
git clone https://github.com/SergeoDev01/voiceover-glass-reel.git \
  ~/.config/opencode/skills/voiceover-glass-reel
```

OpenCode détecte le skill à chaud. Sinon `/reload`, ou :

```bash
opencode service restart
```

### 4.2 Claude Code

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/SergeoDev01/voiceover-glass-reel.git \
  ~/.claude/skills/voiceover-glass-reel
```

### 4.3 Autres agents (format « Agent Skills »)

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/SergeoDev01/voiceover-glass-reel.git \
  ~/.agents/skills/voiceover-glass-reel
```

### 4.4 Vérification

```text
Liste les skills disponibles.
```

Le skill `voiceover-glass-reel` doit apparaître. Chargez-le avec `@voiceover-glass-reel`.

### 4.5 Mettre à jour

```bash
git -C ~/.config/opencode/skills/voiceover-glass-reel pull
```

---

## 5. Installer les MCP utilisés par le skill

| Service | Type | Clé API | Usage |
|---|---|---|---|
| **Pixabay** | MCP local (`pixabay-mcp`) | **oui**, gratuite | clips B-roll, portraits, lieux |
| **thesvg** | MCP local (`@thesvg/mcp-server`) | non | logos de marques (uniquement si nommés à l'oral) |
| **Wikimedia Commons** | API HTTP via `curl` | non | archives photo/vidéo/audio, musique |

Des fichiers d'exemple prêts à copier sont fournis : [`mcp/opencode.jsonc.example`](mcp/opencode.jsonc.example) et [`mcp/claude.mcp.json.example`](mcp/claude.mcp.json.example).

### 5.1 Node.js

```bash
node --version   # v18+
npx --version
```

Sinon : <https://nodejs.org/>.

### 5.2 Pixabay — clé API gratuite

1. Créez un compte : <https://pixabay.com/accounts/register/>
2. Connectez-vous.
3. Ouvrez **<https://pixabay.com/api/docs/>**.
4. Copiez la **clé API** affichée en haut de la page (`12345678-1a2b3c4d5e6f7a8b9c0d1e2f`).
5. La même clé couvre images **et** vidéos.

Limite documentée : **100 requêtes / 60 secondes**. Ne committez jamais la clé.

#### OpenCode

```bash
opencode mcp add pixabay --global \
  --env PIXABAY_API_KEY=COLLEZ_VOTRE_CLE \
  -- npx -y pixabay-mcp@latest
opencode mcp list
```

Attendu : `✓ pixabay  connected`.

**Windows** (si `spawn npx ENOENT`) :

```powershell
opencode mcp add pixabay --global `
  --env PIXABAY_API_KEY=COLLEZ_VOTRE_CLE `
  -- cmd.exe /d /c npx -y pixabay-mcp@latest
```

Configuration manuelle (`~/.config/opencode/opencode.jsonc`) :

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "pixabay": {
        "type": "local",
        "command": ["npx", "-y", "pixabay-mcp@latest"],
        "environment": {
          "PIXABAY_API_KEY": "{env:PIXABAY_API_KEY}"
        }
      }
    }
  }
}
```

Puis, dans le terminal qui lance OpenCode :

```bash
# macOS / Linux (zsh)
echo 'export PIXABAY_API_KEY="COLLEZ_VOTRE_CLE"' >> ~/.zshrc
source ~/.zshrc
```

```powershell
# Windows PowerShell
setx PIXABAY_API_KEY "COLLEZ_VOTRE_CLE"
```

#### Claude Code

```bash
claude mcp add --env PIXABAY_API_KEY=COLLEZ_VOTRE_CLE \
  --transport stdio pixabay -- npx -y pixabay-mcp@latest
claude mcp list
```

Ou `.mcp.json` à la racine du projet :

```json
{
  "mcpServers": {
    "pixabay": {
      "command": "npx",
      "args": ["-y", "pixabay-mcp@latest"],
      "env": {
        "PIXABAY_API_KEY": "COLLEZ_VOTRE_CLE"
      }
    }
  }
}
```

### 5.3 thesvg — logos (aucune clé)

```bash
opencode mcp add thesvg --global -- npx -y @thesvg/mcp-server
opencode mcp list
```

Configuration manuelle :

```jsonc
{
  "mcp": {
    "servers": {
      "thesvg": {
        "type": "local",
        "command": ["npx", "-y", "@thesvg/mcp-server"]
      }
    }
  }
}
```

Claude Code :

```bash
claude mcp add --transport stdio thesvg -- npx -y @thesvg/mcp-server
```

Outils : `search_icons`, `list_variants`, `get_icon_url`, `get_icon`, `list_categories`.

> Les logos ne sont utilisés que pour les marques **réellement prononcées** dans la voix off. Le skill demande votre accord avant de les télécharger et consigne leur licence dans `work/SOURCES.md`.

### 5.4 Wikimedia Commons — aucune clé

```bash
curl -s "https://commons.wikimedia.org/w/api.php?action=query&list=search&srsearch=filetype:image%201950s%20office%20clerk&srnamespace=6&srlimit=3&format=json"
```

Téléchargement avec User-Agent descriptif (obligatoire pour éviter les 429) :

```bash
curl -L -o archive.jpg \
  -A "VoiceoverGlassReel/1.0 (contact: vous@example.com)" \
  --retry 4 --retry-delay 8 --retry-all-errors \
  "https://upload.wikimedia.org/..."
```

Le skill vérifie `content-type`, `content-length`, et compare le **SHA-1** fourni par l'API quand il existe.

### 5.5 Vérifier les outils

```text
/mcps
```

Puis :

```text
Cherche 3 photos d'archives des années 1950 sur Wikimedia et 3 clips sur Pixabay.
```

En **Code Mode** OpenCode V2 :

```js
tools.pixabay.search_pixabay_videos({ query: "1950s office", video_type: "film", per_page: 5 })
tools.thesvg.search_icons({ query: "openai" })
```

Sans Code Mode : `pixabay_search_pixabay_videos`, `thesvg_search_icons`, etc.

---

## 6. Utilisation complète

### 6.1 Le prompt unique

```text
Utilise le skill @voiceover-glass-reel. Voici ma voix off : /chemin/vers/ma_voix.wav

Transforme-la en reel terminé : coupe toutes les pauses et respirations, écris l'histoire visuelle
à partir de ce que je dis, source chaque image gratuitement (photos d'archives Wikimedia Commons,
clips et portraits Pixabay, logos SVG via le connecteur thesvg — rien de payant), et monte-le dans
After Effects. Chaque clip doit être dans un cadre de verre arrondi flottant avec une fine bordure
lumineuse et un léger push-in. Quand j'énumère des points, mets une grille en perspective derrière
et fais changer la couleur de chaque mot-clé au moment où je le dis. Les sous-titres principaux sont
grands et répartis sur l'image, et les cartes de verre tombent exactement sur la coupe. Vérifie,
corrige ce qui cloche, et rends le MP4 1080p final.
```

Options à ajouter dans le même message :

| Option | Effet |
|---|---|
| `Script: <texte>` | les noms et l'orthographe des sous-titres suivent le script |
| `Host: /chemin/webcam.mp4` ou `Host: /chemin/photo.jpg` | fait apparaître une vraie personne dans la TV |
| `Look: couleur, documentaire tech moderne` | change le monde visuel (défaut : N&B archival 1950s) |
| `Accent colour: #FF3B30` | couleur d'accent |

### 6.2 Ce que l'agent fait

1. Crée `<nom>_reel/` à côté de la voix off.
2. `reel.py prep` : convertit l'audio, transcrit, coupe les pauses → `work/voice_cut.wav` + `work/words.json`.
3. Lit tout l'index des mots (`reel.py words`).
4. Écrit `story.json` : monde, casting, shots, textes, listes, effets.
5. Source les assets et vérifie chaque téléchargement (HEAD, taille, SHA-1) → `work/SOURCES.md`.
6. Prépare les chunks hôte, l'alignement, le fond vert, les SFX.
7. `reel.py plan` : checks ; corrige les erreurs.
8. `build.sh` : construit `reel.aep` (~1 min pour 70 s).
9. `render.sh --half` : rendu de contrôle, puis vérification de `work/check.jpg`.
10. `render.sh` : rendu final `out/reel_1080.mp4`.

### 6.3 Le pipeline en commandes

`S` = dossier du skill ; `PROJ` = dossier projet.

```bash
S=~/.config/opencode/skills/voiceover-glass-reel
PROJ=/chemin/ma_voix_reel

# 1. Préparation (coupe les pauses, transcrit)
python3 "$S/scripts/reel.py" prep /chemin/ma_voix.wav "$PROJ"
python3 "$S/scripts/reel.py" words "$PROJ"

# 2. Écrire story.json, puis :
python3 "$S/scripts/reel.py" chunks "$PROJ"   # si shots hôte
python3 "$S/scripts/reel.py" align "$PROJ"    # si shots hôte
python3 "$S/scripts/reel.py" screen "$PROJ/assets/tv_plate.png" "$PROJ"  # si TV
python3 "$S/scripts/reel.py" sfx "$PROJ"
python3 "$S/scripts/reel.py" plan "$PROJ"

# 3. Build AE
"$S/scripts/build.sh" "$PROJ"

# 4. Rendu de contrôle puis final
"$S/scripts/render.sh" "$PROJ" --half
"$S/scripts/bin/frames" "$PROJ/out/reel_half.mp4" "$PROJ/work/check.jpg" 4 3 8 14 21 28
"$S/scripts/render.sh" "$PROJ"
```

Notes :

- `build.sh` refuse de travailler si un autre projet AE a des modifications non sauvegardées. Sauvez-le, jamais de perte de travail.
- `render.sh` utilise `aerender`. Si AE n'est pas dans `/Applications/Adobe After Effects 20*/`, définissez `AE_APP` ou `AERENDER` :

```bash
export AE_APP="Adobe After Effects 2025"
export AERENDER="/Applications/Adobe After Effects 2025/aerender"
```

- `render.sh --half` produit `out/reel_half.mp4` ; le rendu final produit `out/reel_1080.mp4`.
- Si AE 2023 ou antérieur n'a pas de template H.264, `render.sh` retombe sur « High Quality » en `.mov` ; convertissez ensuite en MP4 avec ffmpeg.

### 6.4 Vérifications avant de livrer

Regardez `work/check.jpg` :

- sous-titres lisibles, jamais sur les visages ;
- lèvres du présentateur synchronisées dans la TV ;
- cadres de verre présents ;
- pas de trous noirs ;
- pages non vides.

Corrigez **`story.json`**, puis relancez `plan` → `build` → `render --half`.

### 6.5 Musique et droits

- Par défaut : **pas de musique** (le build l'ignore si `music` est absent).
- Vous pouvez fournir votre piste, ou convertir un fichier audio Wikimedia Commons en MP3.
- Les SFX sont **synthétisés** par `reel.py sfx` : aucune licence.

### 6.6 Révisions

- `plan` et `build` sont déterministes : ils reconstruisent `reel.aep` depuis `story.json`.
- Modifiez `story.json`, pas `reel.aep` à la main — sauf si vous demandez explicitement des retouches manuelles.
- Si vous éditez `reel.aep` vous-même, l'agent ne doit **jamais** rebuild par-dessus sans vous demander.

---

## 7. Structure du dépôt

```text
voiceover-glass-reel/
├── SKILL.md                     # instructions de l'agent
├── PROMPT.md                    # le prompt unique + options
├── README.md                    # ce fichier
├── LICENSE                      # MIT
├── THIRD_PARTY.md               # licences des composants tiers
├── .gitignore
├── references/
│   ├── example_story.json       # story.json d'un reel terminé (70 s)
│   ├── higgsfield.md            # manuel de sourcing gratuit + appendice payant
│   ├── look.md                  # ce que le build dessine (tailles, timings, couleurs)
│   ├── story.md                 # format story.json + règles de shots
│   └── troubleshooting.md       # erreurs et correctifs
└── scripts/
    ├── reel.py                  # prep / words / chunks / align / screen / sfx / plan
    ├── build.sh                 # build AE
    ├── render.sh                # aerender → H.264
    ├── ae_run.sh                # envoie un script à AE et attend
    ├── build_tools.sh           # compile les binaires Swift
    ├── ae/
    │   ├── build_reel.jsx
    │   └── reel_lib.jsx
    └── tools/
        ├── to_wav.swift
        ├── transcribe.swift
        ├── facebox.swift
        ├── frames.swift
        └── transcribe_Info.plist
```

---

## 8. Sources gratuites, licences et attribution

### 8.1 Ordre de sourcing

1. **Clips story** : Pixabay `search_pixabay_videos` (`video_type: "film"`, orientation adaptée, 8–15 s) ou Wikimedia Commons `filetype:video`.
2. **Photos d'archives** : Wikimedia Commons, puis conversion en mouvement Ken-Burns via ffmpeg.
3. **Portraits des cartes** : Pixabay ou Commons.
4. **Logos** : thesvg MCP ou Commons — uniquement si le nom est prononcé, avec votre accord.
5. **Musique** : optionnelle (aucune, la vôtre, ou Commons converti en MP3).

Chaque fichier est vérifié : `content-type`, `content-length`, et SHA-1 quand Commons le fournit. Tout est consigné dans `work/SOURCES.md` et `work/CREDITS.md` (fichier → URL → licence → auteur).

### 8.2 Licences

| Source | Licence | Attribution |
|---|---|---|
| Pixabay | [Pixabay Content License](https://pixabay.com/service/license-summary/) | non obligatoire |
| Wikimedia Commons | **variable selon le fichier** (PD, CC BY, CC BY-SA…) | selon le fichier |
| thesvg | code **MIT** ; logos = marques de leurs propriétaires | usage d'identification |
| After Effects | licence Adobe requise | — |
| SFX | synthétisés par le skill | aucune |
| ffmpeg | LGPL / GPL selon le build | selon le build |

### 8.3 Fichiers open source utilisés

- **pixabay-mcp** — <https://github.com/zym9863/pixabay-mcp> (MIT)
- **thesvg** — <https://github.com/glincker/thesvg> (MIT pour le code)
- **ffmpeg** — <https://ffmpeg.org/> (LGPL/GPL)
- **After Effects** — logiciel commercial Adobe (non open source, prérequis du rendu)

Voir [THIRD_PARTY.md](THIRD_PARTY.md).

### 8.4 Génération payante (optionnelle)

L'appendice de `references/higgsfield.md` documente Seedance via Higgsfield. Il n'est **pas** utilisé par défaut, ne contient aucune clé, et nécessite un compte + des crédits. Il ne s'ouvre que sur demande explicite et après accord sur le coût.

---

## 9. Dépannage

| Symptôme | Cause / solution |
|---|---|
| `ae_run` exit 6 « AE did not start the script » | AE est fermé ou une boîte de dialogue est ouverte. Vérifiez aussi `Réglages Système → Confidentialité → Automatisation` et l'option AE `Allow Scripts to Write Files and Access Network`. |
| `ae_run` exit 4 « still running » | AE est occupé. **N'envoyez pas** une deuxième fois ; attendez et lisez le log. |
| « Another After Effects project with unsaved changes is open » | Sauvez ou fermez votre projet AE. L'agent ne doit jamais le fermer sans sauvegarde. |
| La transcription échoue | Autorisez la **Reconnaissance vocale**. Vérifiez macOS 26+. Pour une autre langue : `REEL_LOCALE=de-DE` avant `reel.py prep`. |
| macOS antérieur à 26 | Fournissez les temps par mot : `{"words":[{"w","start","end"}]}` puis `REEL_WORDS=fichier reel.py prep ...`. |
| `swiftc` introuvable | `xcode-select --install`, puis `./scripts/build_tools.sh`. |
| `permission denied` sur `build.sh` | `chmod +x scripts/*.sh` une fois dans le dossier du skill. |
| Voix coupée trop fort / pas assez | Le seuil est la médiane de la parole − 6 dB (affiché par `prep`). Nettoyez l'enregistrement ou montez le niveau. |
| Lèvres du présentateur désynchronisées | Relancez `reel.py align`. La piste audio du clip hôte doit **être** le chunk (`ffmpeg -map 1:a`). |
| Pas de fond vert trouvé | Re-sourcez la plaque TV (photo CRT frontale + `ffmpeg drawbox` vert), relancez `screen`. Après deux échecs, passez les shots hôte en mode `full`. |
| Trou noir en fin de shot | Le clip est plus court que le shot : choisissez un `in` plus tôt ou un clip plus long. |
| Polices bizarre | Le log du build affiche les polices choisies. Installez SF Pro (Apple) ou modifiez `FONT` dans `build_reel.jsx`. |
| Build lent (> 3 min) | Un build normal fait ~1 min pour 70 s. Fermez les comps lourdes dans la visionneuse. |
| `no H.264 output template` | AE antérieur à 2023 : `render.sh` produit un `.mov` « High Quality » ; convertissez-le en MP4. |
| `spawn npx ENOENT` (MCP, Windows) | `cmd.exe /d /c npx ...` dans la config MCP. |
| Pixabay `400` / `429` | Clé invalide / limite de 100 requêtes par 60 s. |
| Les outils MCP n'apparaissent pas | `/mcps`, puis `opencode service restart` si le service est bloqué. |

---

## 10. FAQ

**Faut-il After Effects ?**
Oui, pour ce skill. Le build et le rendu passent par AE (2024+). C'est la différence avec `talking-head-reels`, qui rend tout en Python/ffmpeg.

**Est-ce que ça marche sur Windows ?**
Pas pour le rendu : AE + transcription Apple + binaires Swift sont macOS. Vous pouvez préparer la transcription et le sourcing sur PC, puis finir sur Mac.

**Puis-je utiliser ma propre voix et mes propres images ?**
Oui. La voix est le point de départ ; vos images sont prioritaires dans le sourcing.

**Combien ça coûte ?**
Le skill et les sources sont gratuits. Il faut une licence After Effects et un Mac compatible macOS 26.

**La musique est-elle obligatoire ?**
Non. Par défaut il n'y en a pas ; vous pouvez fournir la vôtre ou utiliser un fichier Commons.

**Où sont stockées les clés ?**
Dans la config MCP ou, mieux, dans une variable d'environnement (`{env:PIXABAY_API_KEY}`). Aucune clé n'est dans ce dépôt.

**Comment mettre à jour ?**
`git -C ~/.claude/skills/voiceover-glass-reel pull` (ou le chemin OpenCode équivalent).

---

## 11. Licence

Le code et la documentation de ce dépôt sont sous **MIT** — voir [LICENSE](LICENSE).

Les médias, logos, polices et logiciels tiers restent sous leurs licences respectives (voir [THIRD_PARTY.md](THIRD_PARTY.md)). After Effects est un logiciel commercial Adobe, non inclus.

---

**Dépôt :** <https://github.com/SergeoDev01/voiceover-glass-reel>
**Skill associé :** [talking-head-reels](https://github.com/SergeoDev01/talking-head-reels)
