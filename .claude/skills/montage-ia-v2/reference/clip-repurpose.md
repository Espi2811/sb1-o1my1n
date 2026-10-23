---
name: clip-repurpose
description: Transformer une vidéo longue (podcast, interview, stream, sport, action) en clips courts 9:16 / 1:1 / 16:9 — sélection des moments, recadrage sur le locuteur actif, split-screen, time-warp, sous-titres brûlés. Utiliser pour "faire des shorts/reels/TikToks à partir de…".
---

# Clip Repurpose — long → short

Adapté de `louisedesadeleer/clipify` (MIT). Voir `../ATTRIBUTION.md` · point d’entrée : `../SKILL.md`.

## Mode
| Mode | Quand | Détection | Sous-titres |
|---|---|---|---|
| **Dialogue** (défaut) | Podcast, interview, face cam | Whisper → moments | Oui |
| **Action** | Pas de parole, ou couper sur un événement répété (frappe, coup, beat, pas) | Transitoires audio | Non |
| **Hybride** | Mixte | Les deux, intercalés | Sur parties parlées |

## 1A. Candidats dialogue
```bash
ffmpeg -i in.mp4 -ac 1 -ar 16000 a.wav && whisper a.wav --model tiny.en --word_timestamps True --output_format json
ffmpeg -i in.mp4 -af volumedetect -f null -    # pics d'énergie
```
Proposer **3–5 candidats** `[start, end, pourquoi ça marche, titre]`, **10–25 s** (étendre à 30–60 s pour contenu éducatif). Signaux : punchline/réaction ("attends", "non !", rires, jurons), set-up → retournement, auto-dérision autonome, échanges rapides, pics audio. Faire confirmer.

## 1B. Frappes en mode action
- Flux spectral dans la bande 1–6 kHz. Impacts réels ≈ 50–300, bruit ambiant ≈ 5–15.
- `min_flux` 30 (40–50 si faux positifs, 20 si trop peu) ; `min_gap` 0,2 s (batterie) à 0,6–1 s (hache).
- Bande : 200–2000 Hz (sourds), 1000–6000 (bois/métal/verre), 4000–10000 (sifflets, caisse claire).
- Cible : 50–65 frappes pour 45–60 s (~0,9 s/plan). Plan = frappe −0,4 s → +0,5 s.

## 1.5. Vérification visuelle (obligatoire)
Extraire 1 frame à chaque temps candidat (320 px de large). Rejeter si sujet hors cadre/obstrué, doigt sur l'objectif, ou son venant d'ailleurs. **L'audio détecte des sons, pas des événements visibles.**

## 2. Trim
`-c copy` pour l'exploration ; ré-encoder pour la précision à l'image.

## 3. Recadrage 16:9 → 9:16
- 1 locuteur : crop centré (ou suivi visage).
- 2 visages : demander **pan en cut sec** (dialogue rapide) ou **split-screen**.
- Compter les changements de plan : `select='gt(scene,0.3)'` — des ROI fixes échouent à travers les cuts.

### Pan entre visages
1. Frame milieu → ROI bouche+menton de chaque visage `x,y,w,h` (éviter mains, micros) ; vérifier en dessinant les boîtes ; ≤ 2 itérations.
2. Énergie de mouvement par ROI : `crop → format=gray → tblend=all_mode=difference → signalstats` (YAVG) par frame.
3. Timeline locuteur avec **dwell minimum 1,0 s** (les interjections courtes restent sur le locuteur précédent).
4. Source 1920 → bande 608 px (sortie 1080×1920) : `LEFT_X = cx_gauche − 304` (≥ 0), `RIGHT_X = cx_droite − 304` (≤ W − 608).
```bash
ffmpeg -i clip.mp4 -vf "crop=608:1080:'<expr_x>':0,scale=1080:1920:flags=lanczos" \
  -c:v libx264 -preset fast -crf 20 -pix_fmt yuv420p -c:a aac -b:a 192k out_916.mp4
```
Source 4K : downscaler d'abord ou doubler les coordonnées.

### Split-screen
Deux tuiles 1080×960 empilées ; crop ~720×640 autour de chaque visage ; le locuteur actif en haut (overlay `enable=` sur ses segments).

### Time-warp (alternative au trim, garde l'arc)
`setpts=PTS/N` + `atempo` ; 1,5× (mouvement), 2× (reveal punchy), 3–4× (time-lapse). **`atempo` n'accepte que 0,5–2,0** → chaîner `atempo=2.0,atempo=2.0` pour 4×. Couper l'audio si c'est juste de l'ambiance.

## 4. Sous-titres (dialogue)
Re-transcrire **le clip coupé**, pas la source. Styles : *Opus* (gros blanc gras, mot actif jaune), *Karaoké* (chunks de 4 mots, surlignage vert), *Minimal* (Helvetica propre). Brûler en ASS :
```bash
ffmpeg -i out_916.mp4 -vf "subtitles=captions.ass" -c:v libx264 -preset fast -crf 20 -c:a copy final.mp4
```
Détails de style → `captions-animees`.

## 5. Livraison
`<source_dir>/clipify_out/` ; une ligne par clip : nom, durée, pourquoi il marche, chemin. Proposer d'itérer (style, ROI, layout, timing).

## Pièges
- Sources avec sous-titres déjà brûlés : retrouver le master propre (corrélation croisée audio FFT).
- Régler la bande spectrale avant de monter le seuil.
- Annoncer le plan en une ligne, puis agir.
