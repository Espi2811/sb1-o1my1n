---
name: beat-sync
description: Monter sur la musique — grille BPM→frames, offset du premier temps, coupes par phrases, drop, arc de rythme en 4 phases, J/L-cuts en frames, speed ramps, personnalités rythmiques. Utiliser pour clips, pubs, trailers, montages sport/hype, ou tout montage calé sur un morceau.
---

# Beat Sync

Adapté de `iart-ai/motion-design-skills/beat-sync-editing` (MIT) + compléments. Voir `../ATTRIBUTION.md` · point d’entrée : `../SKILL.md`.

## 1. Spec du morceau (avant tout cut)
- **BPM**, **fps**, **offset du temps 1** (ms), frame(s) du **drop**.
- Détection (le skill source ne la précise pas — méthode retenue) :
```python
import librosa
y, sr = librosa.load("track.wav", sr=None, mono=True)
tempo, beats = librosa.beat.beat_track(y=y, sr=sr, units="time")
onsets = librosa.onset.onset_detect(y=y, sr=sr, units="time")
# vérifier à l'oreille : un BPM détecté peut être doublé/divisé par 2
```
- **frames_par_temps = (60 / BPM) × fps** → 120 BPM @30 fps = 15 f ; 128 BPM @30 fps = 14,06 f.
- **Arrondir la position cumulée** (`round(offset_f + n × fpb)`), jamais chaque pas → sinon dérive.
- Tout ancrer sur l'offset du temps 1 ; l'ignorer = cuts "presque justes" mais quelques frames en avance.

## 2. Grille (120 BPM, 30 fps)
| Subdivision | ms | frames | Usage |
|---|---|---|---|
| Noire | 500 | 15 | Grille de coupe principale |
| Croche | 250 | 7,5 | Accent, flash cut |
| Double croche | 125 | 3,75 | Hits, stutters, FX transitoires |
| Mesure (4 temps) | 2000 | 60 | Frontière de phrase/section |

- **Couper sur des phrases de 2, 4 ou 8 temps**, pas sur chaque temps (frénétique).
- Accents sur la grille de croches ; coupes structurelles sur temps/mesure.
- **Le plus gros visuel sur le drop.**

## 3. Placement du cut
- Cut on action, en plein mouvement (rotation, geste, whip), pas au repos.
- Conserver direction et vitesse du mouvement à travers le cut.
- Motion graphics : couper au pic de transformation.

## 4. Transitions
| Type | Réglage | Usage |
|---|---|---|
| Cut sec | offset 0 | **~90 % des coupes** |
| Match cut | tolérance 1–2 f | Forme/position/vecteur partagé |
| J-cut | audio suivant **4–12 f** avant l'image | Dialogue, révélation |
| L-cut | audio courant traîne **12–24 f** | Changement de scène, ambiance |
| Whip / flou de bougé | fondu 6–10 f, flou même direction | Hype, saut de scène |
| Fondu, wipe, luma | — | Passage du temps, douceur ; abus = amateur |

Toute transition non-cut doit être **motivée**. Régler le timing avant d'ajouter une transition. Un fondu ne répare pas un mauvais cut.

## 5. Speed ramp (30 fps)
- f0–20 : 100 % (élan) → f20–28 : easing vers **25 %** → **f28 : impact sur le temps** → f28–40 : retour à 100 %.
- La **frame la plus lente tombe sur le temps**.
- Flou de bougé constant à travers la rampe (sinon stroboscope) ; optical flow / frame blending sous 100 %.
- Implémentation : time-remap easé (NLE/AE) ou `interpolate` easé (Remotion).
- Prérequis : rushes en haute cadence (50/60/120 fps) pour des ralentis propres.

## 6. Arc de rythme
| Phase | Coupe tous les | Plans | Tension |
|---|---|---|---|
| Installation | 8–16 temps | Les plus longs | Basse |
| Développement | 8 → 4 temps | Raccourcissent | Montante |
| Climax | 1–2 temps | Les plus courts + plus gros visuel + ramp | Pic |
| Résolution | Un long hold | — | Relâche |

L'énergie suit la fréquence de coupe ; un rythme uniforme = plat.

## 7. Personnalité → rythme
| Personnalité | Phrase | Transitions | Retime |
|---|---|---|---|
| Playful | 4 temps, syncopé | Match cuts, whips rapides | Ramps légères, rebondies |
| Premium | 8–16 temps | Cut sec, fondu occasionnel | Lentes, douces |
| Corporate | 8 temps réguliers | Cut sec | Minimal |
| Energetic | 1–2 temps au climax | Cut sec, whips, snaps | Agressives sur les temps |

## 8. Son (ajout compendium)
- Riser : **pic sur la frame du drop**, trimmer l'entrée.
- Impact sur le temps ; 2–6 frames de silence avant = contraste max.
- Whoosh sur le cut, pas avant.

## 9. Livrable
Spec du morceau · arc (4 phases, plages de frames, durée moyenne de plan) · **cut list en numéros de frame absolus** (clip, type) · transitions (type + offset audio J/L en frames) · retimes (de→à %, frames, optical flow) · tableau des marqueurs de temps · mapping code (`from`/`durationInFrames`).

Exemple : 128 BPM, 30 fps, spot 30 s, drop au temps 32 → frame ≈ 450 ; installation 0–120 (deux holds de 2 s) ; développement 120–450 (8→4→2 temps) ; climax 450 ; résolution 700–900 (CTA) ; match cut f112 (logo rond → roue) ; ramp 100→25→100 % au drop ; CTA entre sous un L-cut de ~18 f.

## 10. Vérification
- [ ] `frame/fps` comparé aux temps détectés (code) ; piste guide calée à l'image (NLE).
- [ ] Drop, frame la plus lente de chaque ramp, offsets J/L vérifiés contre l'audio.
- [ ] Pas de cut sur chaque temps ; pas de fondu partout ; pas de durée de plan uniforme.
