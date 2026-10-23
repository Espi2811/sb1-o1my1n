---
name: montage-ia-v2
description: Montage vidéo assisté par IA — dérushage de face-cam, long→shorts 9:16, rétention short-form, sous-titres animés, motion design (easing, composition, safe areas), montage sur la musique, Remotion/FFmpeg, sound design, loudness, étalonnage. Utiliser pour planifier un montage, écrire des commandes FFmpeg, coder une animation Remotion, analyser une courbe de rétention ou critiquer un cut.
---

# Montage IA v2

## Routage — lire le fichier de référence adapté AVANT d'agir
| Tâche | Fichier |
|---|---|
| Dérusher un face-cam (silences, fillers, prises) | `reference/cut-rushes.md` |
| Long → clips courts, recadrage 9:16 locuteur actif | `reference/clip-repurpose.md` |
| Structure hook/boucle/payoff d'un short | `reference/short-form-retention.md` |
| Sous-titres mot à mot animés | `reference/captions-animees.md` |
| Langage de mouvement, easing, durées | `reference/motion-direction.md` |
| Mise en page, safe areas, multi-format | `reference/composition-cadre.md` |
| Montage sur la musique, speed ramps | `reference/beat-sync.md` |
| Code Remotion, commandes FFmpeg, exports | `reference/remotion-ffmpeg-pro.md` |

Plusieurs fichiers peuvent s'appliquer (ex. un short = `short-form-retention` + `captions-animees` + `beat-sync`). Ce fichier garde les garde-fous transverses et la philosophie (Murch, cognition, son, étalonnage).

## 0. Garde-fous (failles connues de l'IA)
- **Ne jamais juger le rythme, le jeu ou l'émotion sans voir les frames.** Extraire d'abord :
  ```bash
  ffmpeg -i in.mp4 -vf "fps=2,scale=640:-1,tile=6x5" -frames:v 1 contact_%03d.jpg
  ```
  + transcript horodaté (Whisper `--word_timestamps True`).
- **Ne jamais déduire mouvement ou ordre depuis une frame isolée** (faiblesse temporelle des VLM) : comparer des séquences, raisonner en timecodes.
- **Toujours rendre puis inspecter** un visuel généré (SVG, Remotion, HTML) : `npx remotion still <comp> --frame=N` ou screenshot Chromium → critique → correction.
- **Ne pas inventer de stats de rétention.** Seuls chiffres citables : captions +12 % watch time (Facebook interne), +7,32 % vues YouTube (DDN contrôlé). Sinon : analytics de l'utilisateur.
- **Décisions émotionnelles = proposition, pas verdict.** L'humain tranche (Murch #1–#2 = 74 % du poids).

## 1. Arbitrage d'un cut (Murch)
Émotion 51 > Histoire 23 > Rythme 10 > Eye-trace 7 > Plan 2D 5 > Espace 3D 4. En conflit, sacrifier depuis le bas.

## 2. Placement du cut
1. Sur une **frontière d'événement** (fin d'action, changement de but, sortie de champ).
2. Précédé d'un **indice attentionnel** (regard, geste, son off) — AToCC.
3. **Cut on action** pour masquer, **match cut** pour lier, **smash cut** pour contraster.
4. Respecter 180° / 30° sauf rupture voulue.
5. **J-cut** pour entrer dans une scène, **L-cut** pour les réactions de dialogue.
6. Durées de plans variées en **1/f** (alterner longs/courts), pas une grille fixe. +1 personnage ≈ +1,5 s de plan.

## 3. Short-form
- Frame 0 : action/visage + texte. Jamais de logo.
- Premier cut < 1,5 s. Sous-titres dès la frame 0, hors UI (marges ~ 15 % bas, 10 % côtés en 9:16).
- Pattern interrupts **aux frontières naturelles**, pas toutes les N secondes.
- Analyse : rétention absolue vs relative ; dip = sortie → diagnostiquer (lenteur, CTA, rupture qualité, hors-sujet, pub) ; spike = re-visionnage ; dip récurrent = problème de format.

## 4. FFmpeg — recettes sûres
```bash
# Normaliser VFR -> CFR avant tout xfade/concat
ffmpeg -i in.mp4 -vf fps=30 -af aresample=async=1 -c:v libx264 -crf 18 -preset slow -c:a aac -b:a 192k cfr.mp4
# Trim précis (ré-encodage, pas -c copy)
ffmpeg -ss 00:01:02.500 -i cfr.mp4 -t 4.2 -c:v libx264 -crf 18 -c:a aac clip.mp4
# Crossfade vidéo+audio (offset = durée clip A - durée fondu)
ffmpeg -i a.mp4 -i b.mp4 -filter_complex \
 "[0:v][1:v]xfade=transition=fade:duration=0.5:offset=3.7[v];[0:a][1:a]acrossfade=d=0.5[a]" \
 -map "[v]" -map "[a]" out.mp4
# Loudness 2 passes
ffmpeg -i in.wav -af loudnorm=I=-14:TP=-1:LRA=11:print_format=json -f null -
```
Ne pas utiliser `-fflags +genpts -r` pour "forcer" un framerate.

## 5. Remotion — règles
- Tout dérive de `useCurrentFrame()`. Pas de `useState` temporel, pas de `Date.now()`.
- `random('seed')` au lieu de `Math.random()`.
- `interpolate(..., {extrapolateLeft:'clamp', extrapolateRight:'clamp'})` systématique.
- Easing par défaut : `Easing.bezier(0.16, 1, 0.3, 1)` ; spring sans rebond `damping:200`, UI `damping:20, stiffness:200`, ludique `damping:8`.
- `<OffthreadVideo>`, `<Img>`, `<Audio>`, `delayRender()` pour les données ; polices : uniquement les poids/subsets utilisés ; mesures de texte après chargement.
- Flicker persistant → `--concurrency=1` en diagnostic.

## 6. Sound design
- Layering : 1 layer sub (20–80 Hz), 1 attaque (4–12 kHz), 1 identité (0,5–4 kHz) ; HP/LP croisés.
- Riser : pic sur la frame cible, trimmer l'entrée ; résoudre sur hit ou silence (2–6 frames).
- Whoosh sur le cut ; son du plan suivant 4–12 frames avant l'image.
- Room tone en underlay continu ; dialogues en checkerboard A/B ; foley en stems.
- Livraison : YouTube −14 LUFS / −1 dBTP ; TikTok/Reels −14 à −10 LUFS (vérifier par mesure) ; parole ~−16 LUFS.

## 7. Étalonnage
Pipeline couleur → correction (expo, WB, contraste/pivot) → scopes (parade, waveform, vectorscope) → look → peau en nœud local (teinte avant saturation). Skin line = teinte, pas cible de luma.

## 8. Vidéo générative
1 sujet · 1 action · 1 mouvement caméra par prompt. Mains au repos/fermées. Corriger la scène fautive au lieu de tout régénérer. Lip-sync : CFR, visage frontal, passes séparées.
