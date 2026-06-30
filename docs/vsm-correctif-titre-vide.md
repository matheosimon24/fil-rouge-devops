# Value Stream Mapping — Correctif "titre vide refusé"

**Cas :** Livrer le correctif `POST /tasks` qui accepte un titre fait uniquement d'espaces  
**Début :** Bug signalé, décision prise de le traiter  
**Fin :** Correctif en production, comportement vérifié  
**Unité :** minutes

---

## Carte du flux

```
[Ticket en file]  →  [Reproduction + correctif]  →  [PR en attente]  →  [Revue PR]  →  [Tests CI]  →  [Attente déploiement]  →  [Déploiement]  →  [Vérification prod]
```

---

## Tableau des étapes

| Étape | Description | Travail (min) | Attente (min) | Total étape |
|---|---|---:|---:|---:|
| 1 | Ticket en file d'attente (non prioritaire) | 0 | 480 | 480 |
| 2 | Reproduction du bug + écriture du correctif | 30 | 0 | 30 |
| 3 | PR ouverte, personne ne regarde encore | 0 | 240 | 240 |
| 4 | Revue de la PR par un collègue | 15 | 0 | 15 |
| 5 | Tests CI lancés, attente du résultat | 5 | 8 | 13 |
| 6 | Attente d'un créneau de déploiement hebdomadaire | 0 | 600 | 600 |
| 7 | Déploiement manuel en production | 20 | 0 | 20 |
| 8 | Vérification que le comportement est corrigé en prod | 10 | 0 | 10 |

---

## Totaux

| | Durée |
|---|---:|
| **Temps de travail total** | 0 + 30 + 0 + 15 + 5 + 0 + 20 + 10 = **80 min** |
| **Temps d'attente total** | 480 + 0 + 240 + 0 + 8 + 600 + 0 + 0 = **1 328 min** |
| **Temps total (lead time)** | 80 + 1 328 = **1 408 min** (≈ 23 h 28) |
| **Efficacité du flux** | 80 / 1 408 × 100 = **5,7 %** |

---

## Le goulot identifié

> **Étape 6 — Attente du créneau de déploiement : 600 minutes (10 heures)**

C'est l'étape qui pèse le plus lourd dans le temps total. Ce n'est pas du travail qui prend du temps, c'est une **politique** : l'équipe ne déploie qu'une fois par semaine à un créneau fixe. Un correctif prêt le lundi attend parfois jusqu'au vendredi.

**Nature :** Contrainte organisationnelle (pas un problème d'outillage ou de personnes disponibles).

**Action concrète pour l'éliminer :** Automatiser le déploiement continu (CD). Dès qu'une PR est mergée sur `main` et que la CI est verte, le déploiement se déclenche automatiquement sans intervention humaine. Le créneau fixe disparaît → l'attente de 600 min tombe à ~5 min (durée du pipeline).

---

## Calcul après amélioration (simulation)

En supprimant le créneau unique de déploiement (étape 6 : 600 → 5 min) :

| | Avant | Après CD |
|---|---:|---:|
| Temps d'attente total | 1 328 min | 733 min |
| Temps total | 1 408 min | 813 min |
| Efficacité | 5,7 % | 9,8 % |

Le deuxième goulot qui apparaît est alors l'attente en file du ticket (étape 1 : 480 min).  
Après ça : l'attente de revue (étape 3 : 240 min). On cible un goulot à la fois.

---

## Réponses aux questions guides

**Quelle étape pèse le plus lourd ?**  
L'attente du créneau de déploiement (étape 6, 600 min). C'est de l'attente, pas du travail.

**Si on ne pouvait améliorer qu'une seule étape ?**  
L'étape 6. Elle représente 42 % du temps total à elle seule, et son élimination ne demande aucune compétence supplémentaire — juste d'automatiser ce qu'on fait déjà manuellement.

**Problème d'outillage, de personnes ou de politique ?**  
De politique : le créneau unique de déploiement est une décision organisationnelle, pas une contrainte technique. L'outillage (git, serveur) est déjà en place.

**Comment faire tomber cette attente à presque zéro ?**  
Mettre en place un pipeline CD : merge sur main → CI verte → déploiement automatique. C'est l'objet des TPs suivants du cours.

---

## Pièges évités

- On n'a pas confondu travail (quelqu'un agit) et attente (la tâche ne bouge pas).
- On a utilisé une seule unité (minutes) sur toute la carte.
- On a désigné UN seul goulot, pas plusieurs à la fois.
- On a laissé les chiffres décider : le correctif lui-même (30 min) n'est pas le problème.
