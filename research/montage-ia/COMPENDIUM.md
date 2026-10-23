---
title: "Compendium Montage IA — Failles de Claude, montage pro, psychologie & rétention"
date: "2026-10-22"
---

# Compendium Montage IA

**Périmètre réel de la collecte :** 26 recherches web ciblées (index US), ~230 pages référencées, dont ~60 retenues comme sources exploitables (papiers peer-review, docs officielles Remotion/FFmpeg, presse pro, blogs d'éditeurs). Ce n'est **pas** 2 000 sites : le volume a été sacrifié au profit du tri qualité (une grande partie du web "rétention" recycle les mêmes chiffres non sourcés).

**Légende de fiabilité :** 🟢 peer-review / doc officielle · 🟡 presse pro / praticien reconnu · 🔴 blog vendeur, chiffre non sourcé

---

## 1. Failles de Claude (et des LLM) en vidéo, motion design, graphisme — et solutions

| # | Faille | Cause | Solution opérationnelle | Fiab. |
|---|---|---|---|---|
| 1 | **Ne "voit" pas la vidéo en continu** | Les LLM traitent du texte + des images échantillonnées ; pas de flux temporel natif | Pipeline *frame-sampling* : `ffmpeg -vf "fps=2,scale=640:-1"` → planches contact + transcript horodaté (Whisper) → décisions sur timecodes | 🟢 LAVE (arXiv 2402.10294) |
| 2 | **Raisonnement temporel faible** | Les VLM s'appuient sur les features spatiales intra-frame ; l'info purement temporelle est perdue à l'échantillonnage | Ne jamais déduire mouvement/ordre d'une frame isolée ; fournir des paires/séquences de frames + diff optique ; vérifier l'ordre via timecodes | 🟢 SpookyBench, TOMATO (ICLR 2025), TempCore |
| 3 | **Benchmarks trompeurs** | Seuls 8–33 % des questions des benchmarks vidéo sont réellement sensibles au temps ; beaucoup se résolvent sur une frame ou par prior linguistique | Ne pas se fier aux scores "video understanding" pour juger du rythme d'un montage ; tester sur ses propres rushes | 🟢 TempCore (arXiv 2509.01167) |
| 4 | **Génère du SVG/motion "à l'aveugle"** | Pas de rendu pour s'auto-vérifier | Boucle *render → screenshot → critique → fix* (approche iart-ai/motion-skills, Playwright/Chromium headless) | 🟡 |
| 5 | **Texte-graphique fragile** (kerning, overflow, wrap) | Métriques de police inconnues au moment de l'écriture | Polices explicitement chargées ; `measureText()`/`fitText()` *après* chargement ; safe-zones codées en dur | 🟢 Remotion docs |
| 6 | **Remotion : `Math.random()`** → valeurs différentes entre threads de rendu | Rendu parallèle multi-onglets Chrome | `random('seed')` de `remotion` ; ESLint rule `remotion-deterministic-randomness` | 🟢 |
| 7 | **Remotion : flicker / saccades** | Animation non dérivée de `useCurrentFrame()`, assets non attendus | Tout dériver de la frame ; `<OffthreadVideo>`, `<Img>`, `delayRender()` ; en dernier recours `--concurrency=1` | 🟢 |
| 8 | **Remotion : timeouts polices** | `loadFont()` charge tous poids/subsets | Charger uniquement `weights:['400','800'], subsets:['latin']` | 🟢 |
| 9 | **`interpolate()` non clampé** | Défaut Remotion : extrapolation libre | `extrapolateLeft/Right:'clamp'` systématique | 🟢 |
| 10 | **`spring()` rebondit par défaut** (mass 1, damping 10, stiffness 100) | Défauts physiques | Presets : smooth `damping:200` · snappy `damping:20, stiffness:200` · bouncy `damping:8` | 🟢 |
| 11 | **FFmpeg `xfade` + VFR = désync audio** | xfade exige fps identiques & constants | Normaliser chaque entrée : `-vf fps=30 -af aresample=async=1` avant xfade ; ré-encoder plutôt que `-c copy` pour trims précis | 🟢 FFmpeg-devel/trac #9545 |
| 12 | **`-fflags +genpts -r 30`** "dangereux" | Réécrit les PTS sans recréer les frames | Utiliser le filtre `fps` | 🟡 |
| 13 | **Pas de "frame-perfect" ni de choix créatif sur rushes bruts** | Pas de perception du jeu d'acteur / de l'émotion en continu | IA = cuts structurels (silences, captions, segments) ; humain = choix émotionnels (Murch #1 et #2) | 🟡 |
| 14 | **Vidéo générative : mains, morphing, drift** | Prompts multi-actions | 1 sujet · 1 action · 1 mouvement caméra ; mains au repos/fermées ; corriger la scène fautive plutôt que tout régénérer ; restauration visage par frames | 🔴/🟡 |
| 15 | **Lip-sync qui dérive sur clips longs** | Offset cumulatif par segment ; VFR | Re-encoder en CFR 25/30 ; retirer le B-roll des passes lip-sync ; 2 passes (voix puis lip-sync dédié) ; plan B : doublage audio seul | 🔴 |
| 16 | **Hallucination de chiffres "rétention"** | Le web recycle des stats non sourcées | Ne citer que : FB +12 % watch time captions (interne), DDN +7,32 % vues YouTube captionnées (contrôlé) ; sinon tester ses propres analytics | 🟡 |

---

## 2. Philosophie du montage

### Walter Murch — *Rule of Six* (In the Blink of an Eye) 🟢
| Rang | Critère | Poids |
|---|---|---|
| 1 | Émotion | 51 % |
| 2 | Histoire | 23 % |
| 3 | Rythme | 10 % |
| 4 | Eye-trace (où regarde l'œil) | 7 % |
| 5 | Plan 2D (axe/écran) | 5 % |
| 6 | Continuité spatiale 3D | 4 % |

**Règle d'arbitrage :** si conflit, sacrifier depuis le bas. Ne jamais sacrifier l'émotion pour la continuité. (Les deux premiers = 74 % : un raccord faux mais émotionnellement juste gagne.)

### Hypothèse du clignement
- Murch : couper là où le spectateur clignerait = frontière entre deux idées.
- **Nakano (2009)** : les clignements des spectateurs se synchronisent aux moments "non critiques" (fin d'action, personnage hors champ). 🟢
- **Nuance 2017+** : les clignements semblent liés à la **narration** plus qu'aux cuts ; une étude EEG (n=40) montre que les cuts **réduisent** la fréquence de clignement dans la seconde suivante. 🟢
- **Edit blindness** : les raccords conformes aux règles de continuité sont davantage ratés par l'œil → cécité inattentionnelle, pas clignement. 🟢

### Sally Menke / Thelma Schoonmaker 🟡
- "Compresser le temps dans l'action tout en poussant le personnage à travers la scène" (Menke, apprise chez Scorsese/Schoonmaker).
- Le plan long comme **outil de tension**, pas comme absence de montage (ouverture de *Reservoir Dogs*).
- Monter = choisir **quoi accentuer, combien de temps tenir, quel angle** — pas raccourcir.

---

## 3. Grammaire : cuts, transitions, continuité

| Technique | Définition | Usage |
|---|---|---|
| **J-cut** | Audio du plan suivant avant l'image | Anticipation, suspense, transition douce |
| **L-cut** | Audio du plan précédent qui déborde | Dialogues, réactions, casser le champ/contrechamp |
| **Cut on action** | Couper pendant le mouvement | Masquer le raccord |
| **Match cut** | Forme/mouvement/son identique entre 2 plans | Ellipse, lien thématique |
| **Smash cut** | Coupe brutale entre contrastes | Choc, comédie, réveil |
| **Jump cut** | Même cadre, temps sauté | Énergie YouTube, ellipse assumée |
| **Règle 180°** | Caméra d'un seul côté de l'axe d'action | Cohérence gauche/droite |
| **Règle 30°** | ≥30° (ou changement de valeur) entre 2 plans du même sujet | Éviter le faux jump cut |
| **Eyeline match** | Regard hors champ → ce qui est vu | Point de vue, subjectivité |

**Astuce son-transition :** faire entrer le son du plan suivant avant l'image (J-cut sonore) rend la transition "plus puissante" que n'importe quel effet visuel.

### Plans, coverage, storyboard 🟡
- Workflow conversation : **master large** d'abord → singles en plans moyens → gros plans/réactions → inserts.
- *Taille de plan* (distance) ≠ *type de plan* (hauteur, angle, focale, cadrage).
- **Lined script** = trace de quelle portion est couverte par quel angle ; le storyboard indique **comment les plans se montent**, pas seulement à quoi ils ressemblent.

---

## 4. Psychologie & cognition du spectateur

| Concept | Ce qu'il dit | Application montage | Fiab. |
|---|---|---|---|
| **AToCC** (Tim Smith) | La continuité marche car le spectateur n'attend qu'un petit sous-ensemble des features ; sons off, regards, gestes, mouvement guident l'attention à travers le cut | Placer un **indice attentionnel** (regard, pointage, son off) juste avant chaque cut | 🟢 |
| **Event Segmentation Theory** | Le cerveau découpe le flux en événements quand ses prédictions échouent | Cuts **alignés sur les frontières d'événement** = meilleure mémoire de l'ordre (étude neurones uniques) | 🟢 |
| **Kuleshov** | Le sens naît de la juxtaposition | Effet **réel mais plus faible** que la légende : robuste surtout pour la **tristesse** et chez les spectateurs **expérimentés** ; absent chez les primo-spectateurs | 🟢 Barratt 2016, UEA |
| **Cutting – ASL** | Durée moyenne de plan : ~12 s (1930) → ~2,5 s (2010s) ; structure en **1/f** plus fréquente après 1980 | Varier les durées en 1/f (alternance longs/courts) plutôt qu'un BPM fixe ; **chaque personnage en plus ≈ +1,5 s de plan** | 🟢 i-Perception 2011 |
| **Orienting response** | Les changements visuels captent l'attention bottom-up | Pattern interrupts **aux frontières naturelles**, pas sur une grille fixe ; attention : le rythme rapide ajoute de la charge cognitive | 🟢 Essex (enfants) |

**Aucune preuve** solide d'une cadence universelle "un changement toutes les 3–5 s" : les sources proposent 1,5 s, 5 s, 8–12 s, 30 s. → **Tester contre ses propres courbes de rétention.**

---

## 5. Rétention réseaux sociaux

### Ce qui est solide
- **Sous-titres :** +12 % de temps de visionnage (étude interne Facebook) ; +7,32 % de vues YouTube (DDN, test contrôlé, +13,48 % sur 14 jours) ; 92 % des mobinautes US déclarent regarder parfois sans son (Verizon/Publicis). 🟡
- **YouTube Analytics :** distinguer **rétention absolue** (la courbe) et **relative** (vs vidéos de même durée). Une courbe raide qui bat la moyenne relative est normale pour le format. 🟡
- **Lecture des courbes :** *dip* = sortie (segment lent, CTA verbal, rupture qualité audio/vidéo, hors-sujet vs titre/miniature, pub mal placée) ; *spike* = re-visionnage (incompréhension ou moment fort) ; un dip récurrent au même timecode sur plusieurs vidéos = **problème de format**. 🟡
- **MrBeast production guide (2024, authentifié par 2 ex-producteurs)** : analyse **minute par minute** de la rétention, priorité CTR + rétention. 🟡

### Ce qui est faible (à ne pas répéter comme vérité)
- "Moyenne TikTok 3,75 s", "50–60 % des drop-offs Shorts dans les 3 premières secondes", "70 % de rétention à 3 s sinon reach mort", "73 % des ads e-com échouent en 3 s" → **non sourcés**. 🔴
- Direction cohérente toutefois : **les pertes se concentrent au début** → hook visuel + verbal dans la 1re seconde, pas de logo/intro.

### Hook — checklist actionnable
1. Frame 0 = action ou visage + texte, **jamais** un logo.
2. Question ciblant une douleur précise > ouverture générique.
3. Payoff promis visible (teasing du résultat).
4. Sous-titres dès la frame 0, dans la safe-zone (hors UI TikTok/Reels).
5. Premier cut < 1,5 s.

---

## 6. Sound design, FX, bruitage

### Carte fréquentielle de layering (convention, à ajuster à l'oreille) 🔴/🟡
| Bande | Rôle | Exemple |
|---|---|---|
| 20–80 Hz | Sub / poids | Boom, sub drop |
| 80–500 Hz | Corps | Thump, punch |
| 500 Hz–4 kHz | Identité | Métal, bois, voix |
| 4–12 kHz | Transitoire / attaque | Clic, snap |
| >10 kHz | Air | Shimmer, souffle |

**Règle :** un layer possède le bas, un l'attaque, un l'identité ; le reste = assaisonnement. Filtrer HP sur l'un, LP sur l'autre pour éviter le masquage.

### Riser / whoosh / impact
- **Riser :** son long, pitch/filtre en montée → résout sur un **hit** ou un **silence**. Aligner le **pic** sur la frame cible / le downbeat, puis **trimmer le début**.
- **Whoosh :** court, **sur** le cut pour le masquer ; riser **vers** le cut.
- **Silence de 2–6 frames** avant l'impact = contraste maximal.

### Dialogue & foley 🟡
- **Room tone** : enregistré à chaque setup, **même position micro** ; utilisé en **underlay continu** sous toute la conversation.
- Room tone ("sale", bouche-trous) ≠ ambiance (propre, suggère le lieu).
- **Foley** : passes séparées (pas, tissus, props), spotting session préalable, livraison en **stems**.
- **Checkerboarding** (pratique standard dialogue editing) : alterner les plans/personnages sur des pistes A/B pour traiter EQ/bruit par source et faire des fondus croisés propres.

### Loudness — cibles de livraison
| Plateforme | Intégré | True Peak | Note |
|---|---|---|---|
| YouTube | −14 LUFS | ≤ −1 dBTP | Baisse le trop fort, ne remonte pas le trop faible |
| TikTok / Reels | −14 à −10 LUFS | ≤ −1 dBTP | **Sources contradictoires** → mesurer ses uploads |
| Podcast/talking head | ~−16 LUFS | −1 dBTP | LRA ~11 LU |

```bash
# Normalisation 2 passes EBU R128 (ffmpeg loudnorm)
ffmpeg -i in.wav -af loudnorm=I=-14:TP=-1:LRA=11:print_format=json -f null -
# puis réinjecter measured_I / measured_TP / measured_LRA / measured_thresh :
ffmpeg -i in.wav -af loudnorm=I=-14:TP=-1:LRA=11:measured_I=X:measured_TP=X:measured_LRA=X:measured_thresh=X:linear=true out.wav
```

---

## 7. Motion design

- **Slow in / slow out (easing) = principe n°1** en motion graphics : rien ne bouge à vitesse constante. 🟡
- **Value Graph** : diagonale = vitesse constante ; courbe en S = ease-in/out. **Speed Graph** : colline = accélération puis décélération. 🟡
- **Position** : séparer les dimensions (Separate Dimensions) pour éditer X/Y indépendamment. 🟡
- AE ne copie pas nativement l'easing entre layers (scripts type Flow/EaseCopy). 🟡
- Familles d'easing : Sine < Quad < Cubic < Expo (intensité croissante) ; **Back** = overshoot.
- Les 12 principes Disney (squash & stretch, anticipation, staging, straight-ahead/pose-to-pose, follow-through/overlap, slow in/out, arcs, secondary action, timing, exaggeration, solid drawing, appeal) restent la grille de critique.

**Équivalences code :**
```ts
// Remotion — easing pro par défaut
import {interpolate, Easing, spring} from 'remotion';
const x = interpolate(frame, [0, 20], [-200, 0], {
  easing: Easing.bezier(0.16, 1, 0.3, 1), // "expo out"
  extrapolateLeft: 'clamp', extrapolateRight: 'clamp',
});
const s = spring({frame, fps, config: {damping: 200}}); // sans rebond
```

---

## 8. Étalonnage

1. Valider le pipeline de couleur (IDT → ACEScct → ODT Rec.709).
2. **Correction avant grade** : exposition, balance des blancs, contraste avec pivot.
3. Scopes, pas les yeux : **parade** (équilibre canaux), **waveform** (luma), **vectorscope** (saturation, skin line entre rouge et jaune).
4. Look créatif.
5. Peau en **nœud local** uniquement si nécessaire : qualifier → fenêtre trackée douce → **teinte avant saturation** → bypass pour comparer.
6. La skin line = référence de **teinte**, pas une cible de luminosité/saturation.
7. Nœuds labellisés, groupes pour le travail par scène, stills de référence.

---

## 9. Écosystème de skills IA vidéo identifiés

| Skill / pack | Contenu | Intérêt |
|---|---|---|
| **remotion-dev/remotion** (skills officiels) | Timing, interpolate/spring, polices, assets | Référence technique |
| **digitalsamba/claude-code-video-toolkit** | 13 skills, FFmpeg + Remotion | Pipeline complet ; recommande ré-encodage vs stream copy |
| **iart-ai/motion-skills** | 51 skills motion/kinetic typo/short-form | **Auto-vérification par rendu de frame + screenshot** |
| **heygen-com/skills (video-edit)** | ffmpeg/ffprobe local | Trim, extraction audio |
| **Editframe agent skills** | `editframe-motion-design` | Principes d'animation |
| **Tella skills** (`/cut-video`, `/clipify`) | Montage de rushes existants | Remotion = motion généré ; Tella = footage réel |

⚠️ Je n'ai **pas** téléchargé ni redistribué ces skills tiers (licences variables, code non audité). Le dossier `skills/montage-ia/` contient une **synthèse originale** intégrant leurs bonnes pratiques.

---

## 10. Sources principales

**Peer-review / docs officielles**
- LAVE — LLM agents for video editing: https://arxiv.org/pdf/2402.10294
- TempCore: https://arxiv.org/pdf/2509.01167
- Time Blindness / SpookyBench: https://arxiv.org/html/2505.24867v1
- TOMATO (ICLR 2025): https://proceedings.iclr.cc/paper_files/paper/2025/hash/16ba99f25a235f1100a4014d71d34ad8-Abstract-Conference.html
- Kuleshov replication (Barratt et al., Perception 2016): https://pubmed.ncbi.nlm.nih.gov/27056181/
- Kuleshov & first-time viewers (UEA): https://ueaeprints.uea.ac.uk/62291/
- Cutting et al., i-Perception 2011: https://pmc.ncbi.nlm.nih.gov/articles/PMC3485803/
- Event segmentation & cuts: https://cognitiveresearchjournal.springeropen.com/track/pdf/10.1186/s41235-019-0188-x
- Blink & cuts: https://pmc.ncbi.nlm.nih.gov/articles/PMC6101428 · https://www.doi.org/10.1038/srep43267
- Essex — pacing & attention: https://repository.essex.ac.uk/24861/1/SART%20paper%20%2B%20title%20page%20-%20final%20.pdf
- Remotion flickering: https://v4.remotion.dev/docs/flickering
- Remotion fonts: https://www.remotion.dev/docs/troubleshooting/font-loading-errors
- Remotion randomness: https://v3.remotion.dev/docs/using-randomness
- FFmpeg xfade fps check: https://ffmpeg.org/pipermail/ffmpeg-devel/2020-August/267732.html
- FFmpeg trac #9545: https://ffmpeg.org/pipermail/ffmpeg-trac/2023-June/066302.html

**Presse pro / praticiens**
- No Film School — Murch Rule of Six: https://nofilmschool.com/2016/11/6-rules-good-cutting-according-oscar-winning-editor-walter-murch
- FilmDaft — Rule of Six: https://filmdaft.com/walter-murchs-rule-of-six-the-editors-formula-for-choosing-the-right-cut/
- Scientific American — blink: https://www.scientificamerican.com/blog/literally-psyched/the-innate-irresistibility-of-film
- RogerEbert — Cutting the Basterds (Menke): https://www.rogerebert.com/scanners/cutting-the-basterds
- PremiumBeat — 9 cuts: https://www.premiumbeat.com/blog/9-essential-video-editing-cuts
- Vimeo — J/L cuts: https://vimeo.com/blog/post/j-cuts-l-cuts
- No Film School — essential cuts: https://nofilmschool.com/essential-cuts-every-video-editor-needs-know
- StudioBinder — coverage: https://www.studiobinder.com/blog/film-coverage
- Videomaker — shot lists → storyboards: https://www.videomaker.com/article/c18/10102-from-shot-lists-to-storyboards/
- Tubefilter — MrBeast guide: https://www.tubefilter.com/2024/09/17/mrbeast-internal-production-guide-leaked-key-points/
- 3Play Media — captions studies: https://www.3playmedia.com/blog/studies-find-captions-improve-engagement
- Ableton — high-impact sounds: https://www.ableton.com/fr/blog/learn-how-to-make-high-impact-sounds-for-movies-and-trailers
- Artlist — sound design: https://artlist.io/blog/tips-that-will-take-your-sound-design-to-the-next-level/
- Pixflow — SFX layering: https://pixflow.net/blog/?p=92769
- No Film School — room tone: https://nofilmschool.com/what-is-room-tone
- Mt. Mograph — graph editor: https://mtmograph.com/blogs/tools/how-to-use-the-graph-editor-in-after-effects
- LottieFiles — easing: https://lottiefiles.com/blog/working-with-lottie-animations/easing-editor-give-your-animations-the-flow-they-deserve
- Mixing Light — ACES nodes: https://mixinglight.com/color-grading-tutorials/aces-color-management-using-nodes
- DaVinci Resolve Club — skin: https://davinciresolveclub.com/skin-tone-correction-davinci-resolve/
- practicaly.ai — Remotion + Claude: https://www.practicaly.ai/p/ai-video-editing-remotion-claude
- Tella — Remotion skills: https://www.tella.com/blog/how-to-use-remotion-agent-skills-with-claude-code

**Vendeurs (chiffres à prendre avec prudence)**
- Opus Pro (Shorts/TikTok retention, loudness): https://www.opus.pro/blog/ideal-youtube-shorts-length-format-retention
- Subscribr (retention graphs): https://subscribr.ai/youtube-strategy/youtube-audience-retention-graphs-improve-videos
- Kling — AI video drift: https://kling.ai/blog/fix-ai-video-drift-consistency-guide
- HackerNoon — warped hands: https://hackernoon.com/how-to-fix-warped-hands-and-faces-in-ai-generated-video
- invideo — lip-sync drift: https://invideo.io/faq/why-does-ai-lip-sync-fail-or-drift-on-longer-video-clips/
- CapCut — pattern interrupt: https://www.capcut.com/create/pattern-interrupt-technique-short-form-video
