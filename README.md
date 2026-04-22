# Libheros – Frontend

Ce repository contient la partie frontend du test technique Libheros. L’application permet à un utilisateur de s’inscrire, se connecter, gérer ses listes de tâches et organiser ses tâches quotidiennes via une interface moderne développée avec Vue 3.

## Stack technique

- Vue 3
- Vite
- Vue Router
- TailwindCSS
- Axios

## Fonctionnalités implémentées

L’utilisateur peut :

- créer un compte
- se connecter via authentification JWT
- afficher ses listes de tâches
- créer une liste
- supprimer une liste (avec confirmation)
- sélectionner une liste active
- créer une tâche
- modifier une tâche
- supprimer une tâche (avec confirmation)
- changer le statut d’une tâche (terminée / en cours)
- afficher les tâches terminées dans une section repliable
- afficher le détail d’une tâche dans une sidebar
- naviguer avec une sidebar rétractable

Toutes les actions sont synchronisées avec l’API backend sécurisée.

## Installation du projet

Cloner le repository :

git clone LIEN_DU_REPO_FRONT

Se placer dans le dossier :

cd libheros-front

Installer les dépendances :

npm install

## Lancement du projet

Démarrer le serveur de développement :

npm run dev

Le frontend sera accessible à l’adresse :

http://localhost:5173

## Configuration requise

Le backend doit être lancé sur :

http://localhost:3000

Le frontend communique avec cette API pour toutes les opérations métier.

## Parcours de test conseillé

Créer un compte utilisateur

Se connecter

Créer une liste

Créer une tâche

Modifier une tâche

Changer le statut d’une tâche

Afficher les tâches terminées

Supprimer une tâche

Supprimer une liste

## Auteur

Test technique réalisé par NOM PRENOM