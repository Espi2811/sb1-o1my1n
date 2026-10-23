---
name: cut-rushes
description: Nettoyer un rush parlé (talking head, vlog, tuto, podcast vidéo) — supprimer silences, fillers, hésitations, faux départs et prises ratées, en coupant entre les mots. Utiliser quand l'utilisateur fournit une vidéo face caméra à "dérusher", "jump-cutter" ou "resserrer".
---

# Cut Rushes — dérushage automatique de parole

Adapté de `louisedesadeleer/cut-video` (MIT). Voir `../ATTRIBUTION.md`.

## Entrées
- Chemin vidéo (demander s'il manque).
- Ton : `aggressive` (défaut) · `balanced` · `sentimental` · `documentary`.
- Dossier de travail : `/tmp/cut-rushes/<basename>/` (conservé pour debug).

## 0. Proxy
Créer un proxy 1080p H.264 si la source est HEVC, > 500 Mo ou ≥ 4K. Sonder l'orientation **avant** de scaler.
```bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,codec_name -of csv=p=0 in.mp4
ffmpeg -i in.mp4 -vf "scale=-2:1080" -c:v libx264 -preset fast -crf 20 -pix_fmt yuv420p -c:a aac -b:a 192k proxy.mp4
# portrait : scale=1080:-2
```

## 1. Transcription (texte seulement)
```bash
ffmpeg -i proxy.mp4 -ac 1 -ar 16000 audio.wav
whisper audio.wav --model small.en --output_format json
```
**Les timestamps Whisper ne servent jamais à couper** : il fusionne les bégaiements réels et hallucine des mots aux jonctions.

## 2. Alignement forcé (MFA)
`mfa align --clean corpus/ english_mfa english_mfa aligned/` → tier `words` du TextGrid ; intervalles vides = vrais silences.
- Sur footage plein de prises : MFA peut dériver de 1–6 s → l'étape 4 est obligatoire.
- Un mot court qui dure ≥ ~1 s cache une pause, un raté ou une prise entière → re-vérifier.
- Fallback sans MFA : mots Whisper + `silencedetect`, et le signaler dans le plan.

## 3. Liste de coupes
### 3a. Silences — jamais de seuil dB codé en dur
- Plancher de bruit **F** = 10e percentile RMS (fenêtres 0,5 s) ; niveau parole **S** = 90e percentile.
- Seuil = **F + 6 dB** ; `silencedetect=n=<seuil>dB:d=0.14` (ou `d=0.12` + pad 0,03 s pour "quasi aucune pause").
- Padding : ~0,04 s (silencedetect) / ~0,02 s (MFA).
- Si **S − F < ~8 dB** : ne pas se fier à l'énergie ; utiliser les frontières MFA.

| Gap | aggressive | balanced | sentimental | documentary |
|---|---|---|---|---|
| < 0,25 s | garder | garder | garder | garder |
| 0,25–1 s | → 0,1 s | → 0,3 s | → 0,5 s | garder |
| 1–3 s | → 0,2 s | → 0,8 s | → 1,5 s | → 2 s |
| > 3 s | → 0,2 s | → 0,5 s | → 1 s | → 3 s |

### 3b. Fillers (toujours coupés, pad 50 ms)
EN : um, uh, umm, uhh, er, erm, ah, mhm, hmm. **FR (ajout)** : euh, heu, bah, ben, hein (isolé), mmh.

### 3c. Hedges (mode aggressive, si utilisés comme remplissage — vérifier le contexte)
EN : like, you know, I mean, I guess, kind of, sort of, basically, literally, actually, so, okay so, anyway, honestly.
**FR (ajout)** : genre, en fait, du coup, voilà, tu vois, quoi (final), bon, alors, enfin, disons, franchement, carrément.

### 3d. Faux départs & répétitions
- Même ouverture reformulée en ≤ 3 s → supprimer la 1re tentative + le gap.
- Similarité 3-grammes > 0,8 dans 3 s → marquer l'instance antérieure.
- ≥ 3 mots identiques consécutifs → garder la prise la plus propre.

### 3e. Prises complètes refaites (self-shot)
- Similarité 4-grammes > 0,7 sur fenêtre 60 s, + comparaison des 3–5 premiers mots de phrase.
- **Garder la dernière prise** sauf si elle est ratée. Indices de reprise : longue pause, respiration, clic, "je recommence".
- Lister chaque groupe de prises dans le plan.

### 3f. Contenu protégé
- Fenêtre RMS > F + ~10 dB → **rire / réaction : garder** (soupirs, "wow", souffles = contenu).
- Temps comiques en aggressive : réduire à 0,4–0,6 s, ne pas supprimer.
- Punchline d'un mot : garder la pause avant.

### 3g. Passe de rigueur
1. Scanner les mots gardés : doublons consécutifs, bigrammes dupliqués de part et d'autre d'une coupe, débuts de phrase répétés.
2. Couper à la fin complète du mot + 0,01 s.
3. **Piège OOV** : acronymes/marques absents du dictionnaire MFA disparaissent → si la fin Whisper dépasse le dernier mot MFA de > 0,2 s, prendre la plus tardive.
4. Re-transcrire le résultat avec un 2e ASR indépendant ; chercher doublons et fragments ("pers-") ; corriger ; livrer seulement si propre.

## 4. Vérité terrain de chaque frontière
- Re-transcrire chaque segment gardé dans une fenêtre isolée (`small.en`), démarrant ≥ 0,5 s avant l'attaque attendue.
- Mot touchant le bord de fenêtre = coupé → élargir de ~1 s.
- Padding final : attaque −0,03 s, fin +0,04 à +0,08 s.
- Pauses internes > 0,2 s → ~0,1 s ; garder les respirations voulues ≥ ~0,15 s.
- Coût : ~2 min par 5 min de vidéo.

## 5. Plan + auto-contrôle (même tour que le rendu)
Afficher : durée avant/après, % coupé, nb de segments, coupe médiane, coupes > 3 s, compte fillers/hedges/faux départs, pauses préservées.
- Médiane > 2 s → pas assez serré.
- Médiane < 0,7 s → sur-coupé ; relâcher le seuil 0,25–1 s à 0,3 s.
- Cible aggressive : **40–60 % de compression**. Footage majoritairement fait de prises abandonnées : 65–75 %, médiane 3–5 s, c'est normal.
- Référence : 335 coupes / 460 s, médiane 1,13 s, ~60 % de compression.

## 6. Rendu
**trim + concat, jamais `select`** (`select` désaligne l'audio).
```bash
# filtergraph.txt — une paire par segment gardé
[0:v]trim=start=1.23:end=4.56,setpts=PTS-STARTPTS[v0];
[0:a]atrim=start=1.23:end=4.56,asetpts=PTS-STARTPTS[a0];
...
[v0][a0][v1][a1]...concat=n=N:v=1:a=1[v][a]

ffmpeg -i proxy.mp4 -filter_complex_script filtergraph.txt -map "[v]" -map "[a]" \
  -c:v libx264 -preset fast -crf 20 -c:a aac -b:a 192k out_cut.mp4
```
> 500 segments : extraire chaque segment en chunk, liste concat-demuxer, `-c copy`.

## 7. Livraison
`<source_dir>/cut_out/<nom>_cut.mp4` + récap (durées, %, médiane, chemin). Les révisions re-rendent depuis le proxy et les temps vérifiés.

## Hors scope
Zooms, layouts, motion, b-roll, sous-titres (→ `captions-animees`), upload.
