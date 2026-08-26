# Plateforme de suivi Triactis

Application web qui donne à l'équipe une vue partagée et temps réel de l'avancement des
chantiers : tâches, jalons, événements, progression par projet.

**En ligne :** https://fiexplorer11020.github.io/triactis-suivi-ia/
Accès par code partagé. Aucune installation.

## Ce qu'elle contient au 27 août 2026

**191 éléments**, 50 terminés, 16 jalons, répartis sur six projets.

## Six vues, six usages

| Vue | À quoi elle sert | Quand l'ouvrir |
|---|---|---|
| Aujourd'hui | une seule prochaine action mise en avant, plus ce qui a bougé | en arrivant le matin |
| Focus | ce qui est faisable maintenant, ensuite, plus tard, dépendances comprises | pour choisir sur quoi se mettre |
| Projets | projet, tâche, sous-tâche, avec report automatique d'avancement | pour une revue de fond |
| Kanban | colonnes par statut, couloirs par projet | en réunion d'équipe |
| Gantt | calendrier au jour près, jalons en losange | pour arbitrer des dates |
| Bilan | complétion par mois, répartition par projet | pour un point de pilotage |

## Architecture

- **Front** : un seul fichier HTML autonome, JavaScript et SVG, sans build ni dépendance.
  `triactis-roadmap.html` en est une copie stricte, ancien point d'entrée.
  Toute modification d'`index.html` doit y être recopiée à l'identique.
- **Données** : base partagée, synchronisation temps réel, sauvegarde automatique.
  Sans configuration, l'application bascule en mode local et démarre sur un jeu de démonstration.
- **Déploiement** : pages statiques, tout push sur `main` redéploie, l'adresse ne change pas.

## Alimentation programmatique

Deux points d'entrée exposés à la page :

| Appel | Effet |
|---|---|
| `coworkSnapshot()` | renvoie l'état courant, sans rien modifier |
| `coworkApplyPatch(lot)` | ajoute ou met à jour des éléments, et écrit dans la base partagée |

Le lot est un objet décrivant projets, tags et événements. Un événement dont l'identifiant
existe déjà est mis à jour ; sinon il est ajouté. Le mode `replace` écrase tout et ne doit
servir qu'à restaurer une sauvegarde.

## Précautions

- **Prendre une sauvegarde avant tout lot** : `coworkSnapshot()` puis conserver le résultat.
- **Préfixer les identifiants** d'un lot nouveau, pour ne jamais écraser une ligne existante.
- **Vérifier en navigation privée** après écriture : une modification visible seulement chez
  soi signifie que rien n'a été partagé.
- Attendre que les vraies données soient chargées avant d'écrire, sinon le lot s'applique
  sur le jeu de démonstration.

## Points ouverts

- Le code d'accès est comparé côté page à une constante servie publiquement, avec une fonction
  de hachage non salée : il est retrouvable hors ligne. Le correctif est de restreindre
  l'écriture côté base, pas de masquer le dépôt.
- Le dépôt sert des pages statiques : le passer en privé désactiverait la publication sur un
  compte sans abonnement.
- Trois références à l'adresse actuelle sont écrites en dur : un transfert de propriété doit
  les corriger, sinon la plateforme casse au moment même du transfert.
