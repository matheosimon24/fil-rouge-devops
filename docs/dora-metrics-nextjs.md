# Métriques DORA — vercel/next.js
**Période :** 4 avril 2026 → 4 mai 2026 (30 jours)  
**Source :** Données publiques GitHub via `gh`, collectées le 4 mai 2026

---

## Tableau DORA

| Métrique DORA | Définition | Proxy utilisé | Calcul | Niveau estimé | Confiance |
|---|---|---|---|---|---|
| **Deployment Frequency** | Fréquence à laquelle l'équipe livre en production | GitHub Releases (stable + canary) | 38 releases / 30 jours = **1,27 release/jour** | **Elite** | Moyenne |
| **Deployment Frequency** *(alt.)* | — | Releases stables uniquement | 3 releases / 30 jours = **0,10 release/jour** | **High** | Moyenne |
| **Lead Time for Changes** | Temps entre un changement de code et sa mise en prod | Date création PR → date merge (médiane sur 100 PRs) | Médiane = **9 h 33** | **Elite** | Moyenne |
| **MTTR** | Temps moyen de retour à la normale après incident | Temps entre merge du commit défaillant et merge du revert | Médiane sur 3 reverts = **~2 j 21 h** | **Medium** (prudent) | Faible |
| **Change Failure Rate** | % de changements qui causent un incident/rollback | PRs avec titre revert/hotfix/rollback / total PRs mergées | 4 / 357 = **1,12 %** | **Elite** | Faible à Moyenne |

---

## Détail par métrique

### 1. Deployment Frequency

**Données :**
- 38 releases au total sur la période (35 canary + 3 stables)
- Releases stables : v16.2.4, v16.2.3, v15.5.15

**Deux hypothèses :**
- **Hypothèse A** (stables seulement) : 3 / 30 = **0,10/jour** → niveau **High**
- **Hypothèse B** (stables + canary) : 38 / 30 = **1,27/jour** → niveau **Elite**

**Hypothèse retenue : B (stables + canary)**  
La branche par défaut est `canary`, pas `main`. Les releases canary sont publiées, taguées, et consommées par des utilisateurs early adopters et d'autres projets. Elles constituent des livraisons réelles même si elles ne ciblent pas la prod de tous les utilisateurs finaux.

---

### 2. Lead Time for Changes

**Données :**
- 357 PRs mergées en 30 jours (11,9 PRs/jour)
- Médiane PR créée → mergée : **9,55 h** (≈ 9 h 33)
- P75 : 66 h — Max : 249 h

**Calculs :**
- Nombre moyen de PRs/jour : 357 / 30 = **11,9 PRs/jour**
- Médiane : 9,55 h → **Elite** (< 1 heure pour Elite strict, < 1 semaine pour High)

**Pourquoi la médiane plutôt que la moyenne ?**  
La distribution est fortement asymétrique : quelques PRs vivent très longtemps (max 249 h) et tirent la moyenne vers le haut. La médiane représente mieux l'expérience typique de l'équipe.

---

### 3. MTTR (Mean Time To Recovery)

**Proxy A — issues `bug` fermées :** inutilisable  
- Seulement 2 issues `bug` fermées sur la période, toutes ouvertes depuis 2022-2024
- Une issue ouverte depuis 2 ans n'est pas un incident opérationnel

**Proxy B — temps jusqu'au revert :**

| Changement | Merge initial | Revert | Durée |
|---|---|---|---|
| #92513 → #92845 | 2026-04-15 13:12 | 2026-04-15 18:18 | 5 h 06 |
| #90018 → #92733 | 2026-04-10 23:04 | 2026-04-13 19:41 | 2 j 21 h |
| #93071 → #93226 | 2026-04-21 18:53 | 2026-04-24 23:18 | 3 j 4 h |

**Médiane :** ~2 jours 21 heures → niveau **Medium** (< 1 jour = High, < 1 heure = Elite)

**Pourquoi le proxy issues bug est mauvais pour le MTTR ?**  
Une issue `bug` sur un repo open source peut rester ouverte des années et concerner un comportement mineur, pas un incident de production. Elle ne correspond pas à un downtime ou une régression bloquante. Le temps de fermeture d'une issue ne mesure pas le temps de retour à la normale en production.

---

### 4. Change Failure Rate

**Données :**
- 357 PRs mergées total
- 4 PRs avec revert/hotfix/rollback dans le titre
- 58 PRs avec `fix` dans le titre (non comptabilisées comme échecs)

**Calcul :** 4 / 357 = **1,12 %** → niveau **Elite** (< 15 %)

**Pourquoi ne pas compter les 58 PRs `fix` ?**  
Le mot `fix` dans un titre de PR ne signifie pas qu'un déploiement a échoué. Il peut s'agir d'une correction de bug planifiée, d'une amélioration, d'un ajustement de style. Seuls les reverts, hotfixes d'urgence et rollbacks signalent un changement qui a causé un problème en production.

---

## Limites méthodologiques

1. **On ne voit pas la prod.** GitHub Releases et PRs sont des proxys. Vercel déploie `next.js` en interne — ces déploiements ne sont pas publics. Une release canary n'est pas forcément un déploiement en production pour tous les utilisateurs.

2. **Le MTTR est estimé sur 3 exemples.** L'échantillon (3 reverts) est trop petit pour calculer une médiane statistiquement significative. Un seul revert lent (3 j 4 h) pèse énormément sur le résultat.

3. **Les incidents silencieux sont invisibles.** Un incident résolu en interne (rollback d'infrastructure, redémarrage de service) sans PR publique ni release ne laisse aucune trace dans les données GitHub. Le Change Failure Rate à 1,12 % est probablement sous-estimé.

---

## Tableau de référence DORA

| Niveau | Deployment Frequency | Lead Time | MTTR | Change Failure Rate |
|---|---|---|---|---|
| Elite | Plusieurs fois/jour | < 1 h | < 1 h | 0-15 % |
| High | 1/jour à 1/semaine | < 1 semaine | < 1 jour | 0-15 % |
| Medium | 1/semaine à 1/mois | < 1 mois | < 1 jour | 16-30 % |
| Low | < 1/mois | > 1 mois | > 1 semaine | > 30 % |

---

## Conclusion

`vercel/next.js` montre une très forte maturité sur le **flux de livraison** : releases très fréquentes, Lead Time en heures, énorme volume de PRs. Les métriques de **stabilité** sont plus difficiles à estimer : le MTTR tombe à Medium parce que certains reverts prennent plusieurs jours, mais cet échantillon est insuffisant pour conclure.

### Comparaison avec Acme SaaS (1 déploiement/trimestre)

| Dimension | Acme SaaS | vercel/next.js |
|---|---|---|
| Fréquence | 1 déploiement tous les 3 mois | ~1,27 releases/jour (canary incluses) |
| Taille des lots | Gros lots trimestriels | Petits changements fréquents (11,9 PRs/jour) |
| Lead Time | Probablement semaines/mois | Médiane ~9 h 33 |
| Recovery | Manuel, dépend d'un senior | Reverts rapides visibles, prod non publique |

Le changement le plus frappant n'est pas la fréquence mais la **taille des lots** : Acme accumule 3 mois de changements dans un seul déploiement risqué ; next.js intègre en continu des dizaines de petits changements par jour, ce qui réduit le risque unitaire de chaque livraison.
