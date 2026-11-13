# TP Jour 2 — Réponses aux questions de compréhension

## 1. Quelle est la différence entre une image Docker et un conteneur Docker ?

Une **image** est un modèle figé et immuable : le code de l'application, ses dépendances et son environnement, empilés en couches à partir du Dockerfile. Un **conteneur** est une instance en cours d'exécution de cette image, avec son propre état (processus, fichiers écrits, réseau). Dans ce TP, `mon-service:1.0` est l'image construite avec `docker build` : une seule image a servi à lancer 3, puis 5 conteneurs dans les pods Kubernetes.

## 2. Pourquoi les données de la base sont-elles perdues si l'on supprime le volume, mais pas si l'on supprime seulement le conteneur ?

PostgreSQL écrit ses données dans `/var/lib/postgresql/data`, monté sur le volume `donnees_db`. Ce volume est stocké sur la machine hôte, en dehors du cycle de vie du conteneur. Après `docker compose down` puis `up -d`, le nouveau conteneur a remonté le volume et `/ventes` renvoyait toujours les 2 abonnements. Avec `docker compose down -v`, c'est le volume lui-même qui est supprimé : les données sont perdues et la base repart de zéro.

## 3. Un Deployment Kubernetes déclaré avec replicas: 3 perd un pod. Que fait Kubernetes, et pourquoi parle-t-on d'approche déclarative ?

Kubernetes compare en permanence l'état réel à l'état désiré. Quand il ne reste que 2 pods, il détecte l'écart et en recrée un automatiquement. Pendant le TP, après la suppression du pod `api-deployment-5f8c94dc7f-9nvj2`, un nouveau pod `api-deployment-5f8c94dc7f-m4v7l` est apparu en quelques secondes pour revenir à 3 réplicas, sans aucune intervention.

On parle d'approche **déclarative** parce qu'on décrit dans un fichier YAML *l'état voulu* (« 3 réplicas de l'image mon-service:1.0 ») et non *les commandes à exécuter* pour y arriver. C'est Kubernetes qui se charge d'atteindre puis de maintenir cet état, y compris après une panne.

## 4. Quel est le rôle du Service par rapport aux pods, sachant que ces derniers sont recréés régulièrement avec une nouvelle adresse IP ?

Le Service fournit un **point d'accès stable** (le nom `api-service` et une IP fixe, ici `10.109.219.101`) devant un ensemble de pods qui changent. Il ne vise pas des adresses IP, mais un **label** (`app: mon-service`) : tout pod portant ce label est automatiquement pris en compte, comme le pod `m4v7l` recréé ou les 2 pods ajoutés au scaling, et il est retiré quand il disparaît. Le Service répartit aussi les requêtes entre les réplicas. Les clients n'ont donc jamais besoin de connaître l'IP d'un pod.

## 5. Citez deux limites de Docker seul (sans Kubernetes) que vous avez observées ou apprises ce matin.

- **Pas d'auto-réparation** : si un conteneur plante, Docker seul ne le recrée pas pour maintenir un nombre d'instances voulu, alors que Kubernetes a recréé un pod dès la suppression de `9nvj2`.
- **Une seule machine, sans montée en charge ni répartition de charge** : Docker gère les conteneurs d'un seul hôte. Avec Kubernetes, une commande (`kubectl scale --replicas=5`) a suffi pour passer à 5 instances, réparties derrière un même Service.