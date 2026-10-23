---
name: motion-direction
description: Direction artistique et principes du motion design — définir le langage de mouvement (personnalité, easing, unité de temps, transitions), hiérarchie hero/support/texture, durées, courbes, stagger, anticipation, retenue. Utiliser avant d'animer quoi que ce soit (AE, Remotion, CSS, Lottie, GSAP) ou pour diagnostiquer une anim "cheap/molle/robotique".
---

# Motion Direction

Adapté de `iart-ai/motion-design-skills` : `animation-principles` + `motion-art-direction` (MIT). Voir `../ATTRIBUTION.md`.

**Règle unique : définir le langage de mouvement d'abord, puis animer selon lui.** La direction, c'est surtout de la soustraction : décider ce qui ne bouge pas.

## Workflow (7 étapes)
1. **Brief** : audience, un message, plateforme, durée, 3 adjectifs → une phrase d'intention.
2. **Ton** : une seule case de la matrice.
3. **Spec de langage** (tableau ci-dessous).
4. **Hiérarchie** : hero / support / texture.
5. **Passe de retenue** : couper tout mouvement sans sens ; lister ce qui reste statique.
6. **Direction par plan** : l'intention, pas les keyframes.
7. **Audit de cohérence** : chaque plan passe/échoue contre la spec.

## Matrice ton × énergie (s'engager sur UNE case)
| | Doux | Tranchant |
|---|---|---|
| **Calme** | Luxe, bien-être, éditorial : eases longs, holds généreux | Tech premium, fintech : propre, quasi linéaire, serré mais posé |
| **Cinétique** | Ludique, lifestyle, kids : overshoot, ressort | Sport, hype, gaming : cuts rapides, snaps, overshoot sur le beat |

Mélanger les cases = 1re cause de motion incohérent.

## Personnalités
| Personnalité | Durée | Easing signature | Overshoot |
|---|---|---|---|
| Playful | 150–300 ms | `cubic-bezier(0.34,1.56,0.64,1)` | 10–20 % |
| Premium | 350–600 ms | `cubic-bezier(0.4,0,0.2,1)` | 0 % |
| Corporate | 200–400 ms | `cubic-bezier(0.2,0,0,1)` | 0–3 % |
| Energetic | 100–250 ms | ease-out-expo `cubic-bezier(0.16,1,0.3,1)` | 15–30 % |

## Spec de langage (un choix par ligne, réutilisé partout)
| Propriété | Règle |
|---|---|
| Famille d'easing | Une courbe signature pour ~90 % des mouvements ; max 2 (in pour sorties, out pour entrées) |
| Unité de temps | Durée atomique ; toutes les autres en sont des multiples |
| Famille de transitions | Une seule manière de lier les plans |
| Stagger | Délai fixe + direction fixe |
| Intensité | Budget distance / échelle / overshoot |
| Hold | Immobilité minimale après chaque beat |

Ex. tech premium : décélération, unité 0,4 s (0,2 micro / 0,8 large), match-cut + fondu seulement, cascade 60 ms haut→bas, déplacement ≤ 24 px, scale 0,96→1, overshoot ≤ 2 %, repos ≥ 0,3 s.

## Durées
| Élément | Durée | @30 fps |
|---|---|---|
| Micro-interaction | 100–200 ms | 3–6 f |
| Transition UI | 200–400 ms | 6–12 f |
| Élément hero / plein écran | 400–800 ms | 12–24 f |
| Mouvement caméra cinéma | 800–2000 ms | 24–60 f |
| Entrée / sortie / déplacement / pop marque | 300–500 / 200–300 / 300–400 / 400–600 ms | — |

- Distance ou surface ×2 → durée +30–50 %.
- Lourd = démarre lentement, peu d'overshoot ; léger = snap, rebond possible.
- **Sorties plus rapides que les entrées.**

## Courbes
| Usage | cubic-bezier |
|---|---|
| Entrée (ease-out) | `0.16, 1, 0.3, 1` |
| Sortie (ease-in) | `0.7, 0, 0.84, 0` |
| Déplacement à l'écran (in-out) | `0.65, 0, 0.35, 1` |
| Overshoot marque | `0.34, 1.56, 0.64, 1` |
| Linéaire | **uniquement** boucles continues (spinner, marquee) |

- Un départ lent est bien moins pardonné qu'une fin lente : dans le doute, **décélérer vers le repos**.
- **After Effects** : Easy Ease (F9) = 33 % symétrique, simple point de départ. Rendre asymétrique : influence 80–90 % côté arrivée ; aplatir le Speed Graph vers 0 aux extrémités.

## Springs
- Défaut snappy : `stiffness 300, damping 30, mass 1`.
- Springs pour l'interactif/interruptible ; **durée + easing** pour le chorégraphié calé au timeline (vidéo, sync musique).

## Stagger & rythme
- Listes : 40–80 ms ; grilles denses : 20–40 ms ; **révélation totale ≤ 600–800 ms** (sinon réduire l'offset ou stagger par distance depuis un point focal).
- Ne jamais tout révéler d'un coup.
- **Règle du 1/3 (distance)** : rien ne traverse plus d'~1/3 de l'écran sans keyframe intermédiaire ou changement d'échelle/opacité.
- **Règle du 1/3 (simultanéité)** : avec ≥ 3 éléments, ≤ 1/3 en mouvement actif à la fois.
- Musique 120 BPM : noire = 500 ms, croche = 250 ms ; impacts **sur la grille**. Sans musique : un événement majeur toutes les 500–800 ms.

## Anticipation & follow-through
- Anticipation : contre-mouvement 60–120 ms avant l'action (bouton à 0,95 avant le pop).
- Follow-through/overlap : les éléments attachés se posent 40–80 ms après le corps principal.
- Arcs : légère courbe sur x/y ; la ligne droite sur un objet organique fait mécanique.

## Hiérarchie
- **Hero** : le sujet du plan — mouvement le plus audacieux, le plus lent, le plus easé, posé sur le beat. **Un seul par frame.**
- **Support** : plus petit, plus rapide, s'efface.
- **Texture** : fond, grain, dérive — subliminal, boucles lentes, faible contraste.
- Deux éléments en compétition = échec de direction → rétrograder l'un.

## Retenue
- Pas tout le cadre en mouvement : garder une ancre.
- Pas de transitions empilées (wipe + spin + fade).
- Pas d'animation en boucle sur un texte en cours de lecture.
- Pas d'overshoot sur du sérieux/financier.
- **Un seul "wow" par pièce.**
- Silence et immobilité sont des outils de rythme.

## Diagnostic
| Symptôme | Correctif |
|---|---|
| Raide / robotique | Entrées en ease-out |
| Mou / flottant | Durée −30 %, courbe plus franche |
| Cheap / saccadé | Stagger + durées selon la taille |
| Mécanique malgré bon easing | Anticipation, follow-through, arcs, overlap |

## Audit avant validation
- [ ] Une famille d'easing ; aucun linéaire/défaut parasite.
- [ ] Durées = multiples de l'unité de base.
- [ ] Seule la famille de transitions choisie.
- [ ] Stagger identique partout.
- [ ] Pas de mouvements empilés sans repos.
- [ ] Un hero par frame.
- [ ] Rendu d'au moins 3 frames (début/milieu/fin) inspecté visuellement.
