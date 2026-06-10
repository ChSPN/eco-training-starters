# Backlog heavy-ops

## Contexte du projet

Ce projet simule une page de consultation des remboursements pour un allocataire CAF.
Il représente un service public numérique à fort trafic (plusieurs millions de visites par mois) avec une table volumineuse de données utilisateur, un système d'authentification, et des appels API répétés. L'objectif pédagogique est d'identifier et corriger les anti-patterns d'éco-conception liés au data fetching excessif, à la pagination absente et au polling inutile, dans le cadre d'une démarche d'amélioration continue alignée sur le RGESN et le RWEB.
Cas d'usage cible : consultation de la page de remboursements sur le site de la CAF.


## User story 1

- **Contexte:** En tant qu'allocataire CAF, je veux que l'historique de mes remboursements se charge par tranches de 10 entrées, afin de ne pas attendre le chargement de l'intégralité de mes données à chaque consultation.
- **Objectif:** Réduire de 70% le volume de données chargées au premier affichage
- **Bonne pratique d eco-conception ciblee:** Favoriser la pagination plutôt que le chargement infini (RGESN 4.2 / RWEB BP14)
- **KPI associe:** Taille du payload JSON initial < 50 Ko
- **Repo ou ecran concerne:** Page historique des remboursements — composant tableau paginé — endpoint /api/remboursements
- **Critere de reussite:** Le tableau affiche 10 lignes au chargement initial ; les suivantes se chargent à la demande sans rechargement de page
- **Niveau de priorite:** Haute
## User story 2

- **Contexte:** En tant qu'allocataire CAF, je veux que la page de remboursements ne se rafraîchisse pas automatiquement, afin de ne pas générer des requêtes réseau inutiles pour des données qui ne changent pas en temps réel.
- **Objectif:** 0 requête automatique après le chargement initial de la page
- **Bonne pratique d eco-conception ciblee:** Réduire le nombre de requêtes HTTP (RWEB BP02 / RGESN 6.2)
- **KPI associe:** 0 appel réseau détecté après 30 secondes d'inactivité (DevTools Network)
- **Repo ou ecran concerne:** Page remboursements — service de data fetching — hook ou composant de polling
- **Critere de reussite:** Aucun appel réseau automatique visible dans l'onglet Network de DevTools après le chargement initial
- **Niveau de priorite:** Haute
## User story 3

- **Contexte:** En tant qu'allocataire CAF consultant régulièrement ses remboursements, je veux que les données déjà consultées soient mises en cache navigateur, afin de ne pas re-télécharger les mêmes informations à chaque visite.
- **Objectif:** 100% des ressources statiques et données stables servies depuis le cache lors d'une deuxième visite
- **Bonne pratique d eco-conception ciblee:** Mettre en cache les ressources statiques côté navigateur (RWEB BP49 / RGESN 6.7)
- **KPI associe:** 0 requête réseau pour les données déjà chargées lors d'une deuxième consultation (DevTools Network → from cache)
- **Repo ou ecran concerne:** Page remboursements — configuration headers HTTP cache-control — assets statiques
- **Critere de reussite:** Lors d'une deuxième visite, les ressources statiques et données stables apparaissent "from cache" dans DevTools Network
- **Niveau de priorite:** Moyenne