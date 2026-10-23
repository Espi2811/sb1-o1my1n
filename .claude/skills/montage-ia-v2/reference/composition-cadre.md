---
name: composition-cadre
description: Composition d'image pour motion et vidéo — grilles, point focal, hiérarchie, espace négatif, couches de profondeur/parallaxe, mouvements caméra, entrées/sorties, safe areas par format (16:9, 9:16, 1:1, 4:5), adaptation multi-format. Utiliser pour mettre en page un plan, une scène motion ou décliner un master en plusieurs ratios.
---

# Composition & cadre

Adapté de `iart-ai/motion-design-skills/shot-composition` (MIT). Voir `../ATTRIBUTION.md` · point d’entrée : `../SKILL.md`.

## Grille
- 16:9 : **12 colonnes** (divisible en 2/3/4/6) ; marges 80–120 px ; gouttières 24–32 px ; **unité de base 8 px** (8, 16, 24, 32, 48, 64).
- Largeur colonne = (L − 2×marge − (cols−1)×gouttière) ÷ cols. Ex. 1920, marge 100, gouttière 24 → contenu 1720, colonne ≈ 121,3 px.

## Point focal & hiérarchie
- Sujet principal sur un **point de force des tiers** (~33 %/67 %), pas au centre — sauf symétrie voulue (immobilité, solennité).
- Hiérarchie : **taille > contraste > couleur > position**.
- **Un seul point focal primaire** par frame.
- Espace négatif = élément actif ; ne pas remplir le cadre bord à bord.

## Profondeur & parallaxe
| Couche | Rôle | Vitesse parallaxe | Contraste | Contenu |
|---|---|---|---|---|
| Arrière-plan | Ambiant | 0,1–0,3× | Faible, désaturé/flou | Ciel, dégradé, texture |
| Plan moyen | Secondaire | 0,5–0,7× | Moyen | Environnement du sujet |
| Premier plan | Primaire | 1,0–1,5× | Net sur le sujet | Hero, props proches |

Une seule couche porte le mouvement primaire. Toutes les couches à la même vitesse = "le template entier a bougé".

## Caméra
- **Un mouvement par beat** : push/pull, pan/tilt, follow ou parallaxe — jamais combinés (push + pan + rotate = chaos).
- Push/pull : 800–2000 ms, ease-in-out `cubic-bezier(0.65,0,0.35,1)` ; plus lent = plus cinéma.
- Follow : sujet légèrement décentré, de l'espace devant lui (lead room).
- Premium → pushes lents ; énergique → whip-pans.

## Entrées / sorties
- Suivre le sens de lecture (G→D, H→B en langues LTR).
- Nouveau contenu : entre par la droite ou le bas ; congé : sort à droite/bas ou inverse l'entrée ; approfondir = scale up / push in.
- Par le bord le plus proche ; ease-out à l'entrée (léger overshoot permis), ease-in à la sortie.
- Aucun trajet > 1/3 du cadre sans étape intermédiaire.

## Safe areas (valeurs strictes)
| Ratio | Usage | Marges |
|---|---|---|
| 16:9 | YouTube, web | ~5 % partout (broadcast historique 10 %) |
| 9:16 | Reels, TikTok, Stories | Haut 13–14 %, bas 19–20 %, côtés 6 % (+ rail droit ~120 px) |
| 1:1 | Feed | 6–8 % partout |
| 4:5 | Feed haut | 6 % côtés, 8 % haut/bas |

Texte, logos, visages, CTA dans ces marges.

## Multi-format
- Master en 16:9, point focal dans la bande centrale.
- Pour 9:16 : **ré-empiler verticalement**, ne pas cropper les côtés (crop 16:9→9:16 = titres et logos coupés, contenu sous l'UI).
- Éléments secondaires qui se recomposent par ratio ; le point focal survit à tous les crops.

## Livrable : plan de composition
Spec du cadre (ratios, résolution, fps, grille, unité) · coordonnées du point focal + hiérarchie · carte des couches (rôle, parallaxe) · entrée/sortie par élément (bord, direction, easing, découpage des trajets > 1/3) · le mouvement caméra (propriété, de→à, durée, easing, origine) · safe areas par ratio + plan de ré-empilement vertical.

## Vérification
- [ ] Un point focal sur un point des tiers (ou symétrie assumée).
- [ ] Secondaires non concurrents ; espace négatif réservé.
- [ ] Chaque élément a une couche et une vitesse.
- [ ] Un seul mouvement caméra easé.
- [ ] Contenu critique dans les safe areas **de chaque ratio** ; vertical ré-empilé.
- [ ] Rendu d'un still par ratio inspecté.
