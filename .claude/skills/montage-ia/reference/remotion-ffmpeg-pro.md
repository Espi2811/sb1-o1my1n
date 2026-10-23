---
name: remotion-ffmpeg-pro
description: Règles de production pour vidéo programmatique Remotion et traitement FFmpeg — déterminisme, assets, polices, timing, transitions, durées de scènes, pré-traitement vitesse, exports plateformes, dépannage. Utiliser pour tout code Remotion ou commande FFmpeg/ffprobe.
---

# Remotion + FFmpeg Pro

Adapté de `remotion-dev/skills` et `digitalsamba/claude-code-video-toolkit` (skills `remotion`, `ffmpeg`) — MIT/licences respectives — + docs officielles Remotion/FFmpeg. Voir `../ATTRIBUTION.md` · point d’entrée : `../SKILL.md`.

## Remotion — workflow
- Demande "fais une vidéo" → lancer **Studio** (`npx remotion studio`) et ouvrir la preview **avant** de construire ; la garder ouverte.
- **Ne pas rendre** sans demande explicite d'export/MP4.
- Ne pas écraser des modifications inattendues de l'utilisateur faites hors conversation : demander.
- Consulter la doc à jour (`remotion.dev/docs`) pour toute API non triviale ; mettre à jour via `npx remotion upgrade`.
- Licence : certaines entreprises doivent acheter une licence Remotion pour usage commercial — le signaler.

## Remotion — déterminisme
- Toute valeur animée dérive de `useCurrentFrame()` + `useVideoConfig()`. Pas de `useState`/`useEffect` temporels, `Date.now()`, timers.
- `random('seed')` de `remotion`, **jamais `Math.random()`** (threads de rendu différents = valeurs différentes = flicker).
- Assets : `<OffthreadVideo>`, `<Img>`, `<Audio>` ; données asynchrones via `delayRender()`/`continueRender()`.
- Polices : charger **seulement** les poids/subsets utilisés (`loadFont('normal', {weights:['400','800'], subsets:['latin']})`) ; `measureText`/`fitText` **après** chargement.
- Flicker persistant : diagnostiquer avec `--concurrency=1`.

## Remotion — timing
```ts
interpolate(frame, [0, 20], [0, 1], {extrapolateLeft: 'clamp', extrapolateRight: 'clamp',
  easing: Easing.bezier(0.16, 1, 0.3, 1)});
spring({frame, fps, config: {damping: 200}});              // sans rebond
spring({frame, fps, config: {damping: 20, stiffness: 200}}); // UI snappy
spring({frame, fps, config: {damping: 8}});                  // ludique
```
- `interpolate` + easing pour le chorégraphié calé à la musique ; `spring` pour l'organique.
- Sortie d'un spring (0–1) → `interpolate` vers n'importe quelle plage.
- Convention : **30 fps → frames = secondes × 30**.

## Remotion — transitions (`@remotion/transitions`, `TransitionSeries`)
| Durée | Frames @30 |
|---|---|
| Rapide | 15–20 |
| Standard | 30–45 |
| Dramatique | 50–60 |
| Glitch | 20–30 (doit surprendre) |
| Light leak | 45–60 (doit balayer) |

Usages : glitch = tech/cyber · RGB split = moments énergiques · zoom blur = CTA à fort impact · light leak = célébration chaude · clock wipe = temps/ludique · pixelate = rétro · checkerboard = révélation structurée.

## Durées de scène types (explainer / démo produit)
Titre 3–5 s · vue d'ensemble 10–20 s · démo 10–30 s · stat 8–12 s · crédits 5–10 s · intro sting 3–5 s · end screen YouTube 20 s (laisser les zones d'éléments cliquables). **Voix off ≈ 150 mots/min** (FR : ~140–160) et c'est elle qui pilote le timing.

## Vitesse : Remotion vs FFmpeg
`playbackRate` de Remotion doit être **constant**. Vitesse variable → pré-traiter en FFmpeg.
```bash
# facteur pour faire tenir un rush de D s dans une scène de T s : k = D / T
ffmpeg -i in.mp4 -filter_complex "[0:v]setpts=PTS/1.5[v];[0:a]atempo=1.5[a]" -map "[v]" -map "[a]" fast.mp4
# atempo ∈ [0.5, 2.0] par instance → chaîner : atempo=2.0,atempo=1.5 pour 3×
```

## FFmpeg — recettes
```bash
# GIF -> MP4 compatible (dimensions paires)
ffmpeg -i in.gif -movflags +faststart -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" out.mp4
# Letterbox vers 1920x1080
ffmpeg -i in.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" out.mp4
# Crop pour remplir
ffmpeg -i in.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=increase,crop=1920:1080" out.mp4
# Trim précis : ré-encoder (le stream copy peut perdre des frames aux points de coupe)
ffmpeg -ss 12.5 -i in.mp4 -t 4 -c:v libx264 -crf 18 -c:a aac out.mp4
# VFR -> CFR (avant xfade/concat) — jamais -fflags +genpts -r
ffmpeg -i in.mp4 -vf fps=30 -af aresample=async=1 -c:v libx264 -crf 18 -c:a aac cfr.mp4
# Fondus
ffmpeg -i in.mp4 -vf "fade=t=in:st=0:d=0.5,fade=t=out:st=9.5:d=0.5" -af "afade=t=in:d=0.5,afade=t=out:st=9.5:d=0.5" out.mp4
# Inspecter
ffprobe -v error -show_entries format=duration:stream=codec_name,width,height,r_frame_rate,avg_frame_rate -of json in.mp4
```
| Qualité | CRF x264 |
|---|---|
| Archive | 14–16 |
| Production | 18 |
| Preview web | 23 |
| Brouillon | 28 |

## Exports plateformes
```bash
# YouTube / Shorts
ffmpeg -i in.mp4 -c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p -profile:v high -movflags +faststart \
  -c:a aac -b:a 192k -ar 48000 yt.mp4
# Web embed WebM
ffmpeg -i in.mp4 -c:v libvpx-vp9 -crf 32 -b:v 0 -c:a libopus -b:a 128k web.webm
```
Audio : −14 LUFS intégré, ≤ −1 dBTP (loudnorm 2 passes, voir `montage-ia`).

## Dépannage
| Symptôme | Cause / fix |
|---|---|
| "width not divisible by 2" | `scale=trunc(iw/2)*2:trunc(ih/2)*2` |
| Ne lit pas dans le navigateur | `-pix_fmt yuv420p -movflags +faststart`, profil high |
| Désync audio après changement de vitesse | `atempo` chaîné, vérifier fps constant |
| Fichier trop lourd | Monter CRF, `-preset slower`, baisser résolution |
| Sortie vide silencieuse | Vérifier `-map`, durées de trim, `ffprobe` du résultat |
| xfade décalé | Entrées à fps identiques et constants |

## Vérification
`npx remotion still <comp> --frame=N` sur début/milieu/fin + `ffprobe` du MP4 final (résolution, fps, durée, codecs, loudness).
