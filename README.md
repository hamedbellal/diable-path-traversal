# diable-path-traversal
Conteneur Docker vulnérable pour Path Traversal - Projet DIABLE Master 2 DSI

---
title: "Path Traversal (Directory Traversal)"

tag: "File System / LFI"

difficulty: "Facile"

goal: "Accéder à des fichiers système non autorisés, voler des données sensibles, évoluer vers LFI/RCE."

fix: "Validation stricte des chemins, whitelist des fichiers autorisés, utilisation de basename(), contexte sécurisé."

---

# Théorie

## Qu'est-ce que l'attaque ?

Path Traversal (ou Directory Traversal) est une vulnérabilité qui permet à un attaquant d'accéder à des 
fichiers et répertoires situés en dehors du répertoire autorisé de l'application web, en manipulant 
les chemins de fichiers via des séquences spéciales comme `../`.

## Pourquoi elle existe ?

Elle apparaît lorsque l'application web utilise des entrées utilisateur (comme des paramètres GET/POST) po
ur construire des chemins de fichiers **sans valider ni normaliser** ces entrées. L'application fait une 
confiance excessive aux données externes.

## Dans quels cas réels elle apparaît ?

- Applications de téléchargement de fichiers
- Galeries d'images/documents
- Systèmes de gestion de contenu (CMS)
- Interfaces d'administration
- Endpoints API qui servent des fichiers

## Exemple simple

Une application vulnérable construit un chemin comme ceci :
---
$file = $_GET['file'];  // "cv.pdf"
$path = "/uploads/" . $file;
readfile($path);
----

Un attaquant peut injecter :

../../../etc/passwd

Résultat : l'application lit /etc/passwd au lieu d'un fichier dans /uploads/.

Objectif pédagogique : comprendre que le contrôle d'accès aux fichiers doit être basé 
sur des listes de permissions explicites, et non sur des chemins fournis par l'utilisateur.


# Lab
## Objectif du lab

Observer comment une entrée utilisateur (paramètre de fichier) est utilisée pour accéder au système de fichiers, et réussir à lire des fichiers sensibles en dehors du répertoire autorisé.
Règles

- Pas de scanners automatiques

- Comprendre la structure de répertoires attendue

- Tester progressivement les séquences de traversal

- Documenter chaque fichier sensible découvert

## Accès

    URL : http://[IP]:[PORT]/index.php

    Paramètre vulnérable : file

    Répertoire autorisé : /var/www/html/uploads/

    Indice : Observer les messages d'erreur pour comprendre la structure des dossiers


## Débrief
### Pourquoi ça a fonctionné ?

Parce que l'application a construit un chemin de fichier en concaténant directement l'entrée utilisateur avec un répertoire de base, sans vérifier que le chemin final se trouve bien dans le répertoire autorisé.
Où était la vulnérabilité ?

Typiquement dans une zone du code qui fait :

    $user_file = $_GET['file'];
    $full_path = "/var/www/html/uploads/" . $user_file;
    readfile($full_path);

Ou pire :
    
    include($_GET['page'] . '.php');

