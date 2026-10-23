---
name: captions-animees
description: Sous-titres animés mot par mot (style TikTok/karaoké) à partir d'un audio ou transcript — timestamps réels, pagination, pop, mot actif, safe area, burn-in + SRT/VTT. Utiliser pour toute demande de sous-titres dynamiques, captions, karaoké.
---

# Captions animées

Adapté de `iart-ai/tiktok-video-skills/caption-animation` (MIT). Voir `../ATTRIBUTION.md`.

## Règles d'or
- **Timestamps mot à mot réels** (Whisper/AssemblyAI). Ne jamais répartir une phrase uniformément sur sa durée. Si on n'a que des cues par phrase → re-transcrire.
- **Lisibilité > décoration** : lu sans son, sur téléphone.
- Voix re-enregistrée → **re-transcrire**, ne pas décaler à la main.

## Pipeline
1. Transcrire : Whisper `medium.en` (~2× plus rapide que `large`, suffisant sur voix propre ; `medium`/`large-v3` pour le français) via `@remotion/install-whisper-cpp` ou CLI.
2. Normaliser : `{ text, startMs, endMs }[]` (SRT `HH:MM:SS,mmm` → ms).
3. Paginer : `createTikTokStyleCaptions({captions, combineTokensWithinMilliseconds})`
   - 200–500 ms → vrai mot à mot ; 1000–1500 ms → 2–3 mots par page.
   - 1–4 mots affichés à la fois.
4. Animer pop + mot actif.
5. Layout safe area.
6. Exporter MP4 brûlé **et** SRT/VTT depuis les mêmes tokens.

## Animation (Remotion)
```tsx
const enter = Math.round((w.startMs / 1000) * fps);
const s = spring({frame: frame - enter, fps, config: {damping: 12, mass: 0.6}});
const scale = interpolate(s, [0, 1], [0.6, 1]);
const opacity = interpolate(s, [0, 1], [0, 1], {extrapolateRight: 'clamp'});
const tMs = (frame / fps) * 1000;
const active = tMs >= w.startMs && tMs < w.endMs;
const color = active ? '#FFE45E' : '#FFFFFF'; // transition ~80 ms linéaire
```
- **Une seule** couleur d'accent ou une boîte pleine derrière le mot actif. Jamais animer la couleur de tous les mots en même temps.

## Animation (CSS, hors Remotion)
```css
.w{opacity:0;display:inline-block;animation:pop .26s cubic-bezier(.34,1.56,.64,1) forwards}
@keyframes pop{from{opacity:0;transform:translateY(.18em) scale(.7)}to{opacity:1;transform:none}}
.w.active{color:#FFE45E}
/* animation-delay = startMs de chaque mot */
```

## Typo & layout 9:16 (1080×1920)
| Paramètre | Valeur |
|---|---|
| Police | Sans grasse/extra-grasse : Montserrat, Inter, Helvetica |
| Taille | 56–80 px, **min 45 px** |
| Contour | 2–6 px noir plein |
| Ombre | Portée douce, en secours du contour |
| Casse | Majuscules ou casse phrase, remplissage contrasté |
| Position verticale | Bande centrale, **~62–70 % depuis le haut** |
| Bas | Libérer au moins les ~380 px du bas (≈ 20 %) |
| Côtés | Dans les 80 % centraux de la largeur |
| Lignes | Max 2 ; 4–7 mots/ligne |

Le skill source se contredit sur la marge basse (15 % vs 20–35 %) → on retient ~20 %.

## Burn-in vs sidecar
- **Brûlé (open captions)** : indispensable sur TikTok/Reels, stylable ; ne suffit pas pour l'accessibilité.
- **SRT/VTT (closed)** : accessibilité + SEO ; souvent ignoré par les réseaux.
- → Livrer **les deux**.

## Vérification
1. Stills : frame 0, une frame milieu avec mot actif, dernière frame — avec les vraies props.
2. Le mot surligné correspond au temps de la frame (`frame/fps` ∈ [startMs, endMs]).
3. Contour/ombre tiennent, pas de clipping, dans la safe area à chaque frame vérifiée.
4. Rendu piloté par la frame (pas de `Date.now()`/`Math.random()`/timers).
5. MP4 + SRT/VTT émis.
