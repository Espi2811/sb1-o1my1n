---
name: short-form-retention
description: Structurer et monter une vidéo verticale courte (TikTok, Reels, Shorts, 7–60 s) comme un système de rétention — hook, boucle ouverte, interruptions, payoff, loop, safe zones 9:16. Utiliser pour écrire, monter, auditer ou générer (Remotion) un short.
---

# Short-Form Retention

Adapté de `iart-ai/tiktok-video-skills/short-form-video` (MIT) + recherche du compendium. Voir `../ATTRIBUTION.md`.

**Principe :** on monte la **courbe de rétention seconde par seconde**, pas la beauté. Un montage simple qui garde la courbe haute bat un montage léché qui perd les gens.

## Contrat narratif
- **Une idée par vidéo.** Si la leçon ne tient pas en une phrase, ce n'est pas prêt ; une 2e idée = une 2e vidéo.
- **Ouvrir une boucle dans les ~3 premières s, la fermer en dernier.** Ne jamais répondre trop tôt.
- **Zéro temps mort** : tout beat qui ne fait pas avancer l'idée ni ne relance l'attention est coupé.

| Beat | Temps (short 20 s) | Rôle | Échec typique |
|---|---|---|---|
| Hook | 0–1 s | 1 image + 1 phrase = boucle ouverte | Swipe < 3 s |
| Setup | 1–4 s | Pourquoi regarder : promesse, enjeu, manque | — |
| Body | 4–17 s | Valeur en beats, 1 idée/beat | Milieu plat |
| Payoff / loop | 17–20 s | Fermer la boucle ou renvoyer à la frame 0 | Pas de re-visionnage |

Pour 60 s : on allonge le **body**, jamais le hook (reste à ~1 s).

## Grammaire du hook (1re seconde)
| Type | Forme | Pour |
|---|---|---|
| Boucle ouverte | Affirmation dont la réponse arrive plus tard | La plupart des contenus |
| Promesse directe | "3 façons de X en 30 s" | Éducatif, listicle |
| Pattern interrupt | Snap-zoom, whip-pan, décalage visuel dès la frame 1 | Divertissement |
| Négation | "Arrête de faire X" / "Tu te trompes sur X" | Opinions fortes |

- Texte du hook à l'écran **dès la frame 1** (une majorité regarde sans son).
- Le visuel le plus fort **est** la frame 1. Pas de fondu depuis le noir, pas de logo.

## Interruptions & rythme
- Types : cut sec, zoom punch, insert b-roll, SFX, pop de caption, changement d'angle.
- **Un changement visuel toutes les ~2–4 s**, intervalles **irréguliers** (un métronome = ennui). Ex. cuts à 0 · 1,2 · 3,0 · 5,4 · 8,0 · 11,2 · 14,0 s.
- ⚠️ Aucune étude ne valide une cadence universelle : placer les interruptions **aux frontières naturelles** et valider sur les analytics.
- Zoom punch : scale 1 → 1,12, origin 50 % 45 %, spring `damping:12, stiffness:200`.
- Ne jamais construire le body avec des prises molles.

## Sous-titres
Obligatoires ; 4–7 mots/ligne max 2 lignes, fort contraste, dans la safe zone. Style → `captions-animees`.

## Boucle & re-visionnage
- Boucle propre de 8 s vue 3 fois = 300 % de view-through.
- **Raccord image** : dernière frame = première (composition, position, couleur).
- **Raccord phrase** : la fin enchaîne grammaticalement sur la phrase du hook.
- Durée idéale de boucle : ~7–15 s.

## Safe zones 9:16 (1080×1920) — valeurs strictes retenues
| Zone | Réserver |
|---|---|
| Haut | ~240 px (12–14 %) — profil, son, onglets |
| Bas | ~380 px (≈ 20 %) — légende, CTA, hashtags, barre audio |
| Droite | ~120 px — rail like/commentaire/partage |
| Gauche | ~65 px (6 %) |
| **Boîte sûre universelle** | centre ~900×1300 : hook, sujet, sous-titres |

Réserver le bas même s'il paraît vide dans l'éditeur.

## Chiffres : ne citer que les solides
Le skill source cite "87 % décident en 3 s", "70 %+ de view-through", "58 % vs 41 % avec interruptions toutes les 4 s" — **non sourcés** : à présenter comme heuristiques, jamais comme faits. Solides : captions +12 % watch time (Facebook interne), +7,32 % vues YouTube (DDN, contrôlé).

## Génération Remotion
- `<Composition>` + schéma zod + `defaultProps` ; sortie 1080×1920.
- Tout dérive de `useCurrentFrame()` — pas de `Date.now()`, `Math.random()`, timers.

## Vérification (avant livraison)
```bash
npx remotion compositions                                   # durationInFrames, fps
npx remotion still Short out/f-hook.png --frame=10 --props=props.json
npx remotion still Short out/f-mid.png  --frame=<N> --props=props.json
npx remotion still Short out/f-end.png  --frame=<L-1> --props=props.json
npx remotion render Short out/short.mp4 --props=props.json
```
Checklist :
- [ ] Hook lisible instantanément, visuel le plus fort en frame 1, pas de fondu.
- [ ] Boucle/promesse posée avant ~4 s.
- [ ] Interruption ~2–4 s, intervalles inégaux, pas de milieu plat.
- [ ] Sous-titres présents, 4–7 mots, dans la boîte sûre à chaque frame vérifiée.
- [ ] Frame 0 et dernière frame raccordent (ou phrase-boucle).
- [ ] Rendu avec les **vraies props**, pas seulement les défauts. En batch : valider 1 sujet en stills avant de tout rendre.

## Audit post-publication
Rétention absolue vs relative ; dip = sortie (lenteur, CTA verbal, rupture qualité, hors-sujet vs titre) ; spike = re-visionnage ; dip récurrent au même timecode sur plusieurs vidéos = défaut de format.
