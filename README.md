# Template de mise en production et maintenance d'un site avec Stack Django Vue

## Stack technique

Outils open-sources

- frontend:
  - Serveur nginx
  - framework Vue
- backend: framework Django

Outils externes potentiellement payants

- Mailgun pour les mails
- AWS S3 pour la sauvegarde de la BDD
- Rollbar pour le suivi des bogues en production

## Les différents rôles/playbook

Pour un nouveau serveur ou projet, ils sont à lancer dans cet ordre.

- `bootstrap` : gère les accès en ssh. À lancer en premier sur un nouveau serveur.
  Lancer avec `ansible-playbook bootstrap.yml`.
  N'est pas nécessaire en cas d'ajout d'un projet sur le même serveur.

- `base` : installe les dépendances devops du projet. Configure les mises à jour
  automatiques et les roulements de log nginx
  N'est pas nécessaire en cas d'ajout d'un projet sur le même serveur.

- `backend` : télécharge le code back, installe les dépendances du backend et paramètre
  `supervisord`.

- `frontend` : télécharge le code front, installe les dépendances du backend et paramètre
  `nginx`.

## Procédure de maintenance

La maintenance est toujours faite via Ansible. Il est donc
nécessaire d'installer Ansible sur son PC, en version 2.9.

Pour pouvoir utiliser Ansible, il faut avoir la clé du
"vault" qui contient des informations sensibles.
Cette clé peut être récupérée dans le Lastpass de l'entreprise Telescoop.
Elle doit être copiée en local, à la racine de ce dépôt, dans un fichier
nommé `vault.key`.

Installation du serveur ou mise à jour de tout :

`ansible-playbook all.yml`

### MAJ du frontend

`ansible-playbook frontend.yml`

### MAJ du backend

`ansible-playbook frontend.yml`

### BDD et sauvegardes

## Adapter ce template pour un nouveau projet

Commencer par installer, si nécessaire, [`pre-commit`](https://pre-commit.com/)
et l'activer `pre-commit install`. Cela permet d'avoir des vérifications avant
chaque commit.

- modifier les variables dans `group_vars/cross_env_vars.yml`, notamment :
  - le port ssh
  - les utilisateurs et leurs clés ssh publiques
- générer des identifiants Mailgun, AWS S3 et Rollbar pour le projet.
- créer un fichier `vault.key` avec le texte `CHANGE_ME`: `echo CHANGE_ME > new_vault.key`
- modifier la clé du coffre-fort Ansible avec une clé générée aléatoirement :
  - echo CHANGE_ME > vault.key  # CHANGE_ME est la clé actuelle du coffre-fort
  - `openssl rand -base64 40 > new_vault.key`
  - `ansible-vault rekey --new-vault-password-file new_vault.key  group_vars/all/cross_env_vault.yml`
  - `mv new_vault.key vault.key`
  - `ansible-vault view group_vars/all/cross_env_vault.yml`
- modifier les valeurs du vault: `ansible-vault edit group_vars/all/cross_env_vault.yml`

## TODO

- Comprendre et adapter dans `base` les règles `iptables` et `RAID`.
