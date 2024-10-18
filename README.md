# Automatisation-de-la-production

Voici le code dans le fichier ci.yml en plus des expliquations :

Le workflow est conçu pour exécuter des tests PHPUnit chaque fois qu'il y a un nouveau commit sur la branche main.
Il installe le code source, configure l'environnement PHP, installe les dépendances nécessaires, puis exécute les tests pour s'assurer que le code fonctionne correctement.
L'objectif est d'avoir une intégration continue afin de détecter les erreurs de code rapidement après chaque modification.
```
name: CI - PHPUnit Tests

on:
  push:
    branches:
      - main
```
//// Spécifie les événements qui déclenchent ce workflow. Ici, le workflow s'exécute lorsque des modifications sont poussées vers la branche main.

```
jobs:
  test:
    runs-on: ubuntu-latest
```
//// runs-on: Spécifie l'environnement sur lequel ce job sera exécuté, ici ubuntu-latest, qui est une machine virtuelle avec le dernier Ubuntu disponible.
    
```
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
```
//// Cette étape utilise l'action actions/checkout@v3 pour récupérer le code du dépôt sur la machine virtuelle. C'est essentiel pour que les autres étapes aient accès aux fichiers du projet.

```
    - name: Set up PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.1' 
        extensions: mbstring, dom, curl, gd, sqlite3
        coverage: none
```
//// Cette étape configure PHP en utilisant l'action shivammathur/setup-php@v2.
//// php-version: Définit la version de PHP à utiliser, ici 8.1.
//// extensions: Installe les extensions PHP nécessaires (mbstring, dom, curl, gd, sqlite3).
//// coverage: Défini à none, cela signifie qu'aucun outil de couverture de code n'est configuré.
```
    - name: Install Composer dependencies
      run: composer install --prefer-dist --no-progress --no-suggest
```
//// Cette étape exécute la commande composer install pour installer les dépendances PHP du projet définies dans le fichier composer.json.
//// --prefer-dist: Préfère télécharger les archives des paquets (plus rapide que le clonage git).
//// --no-progress: Désactive l'affichage des informations de progression pour une sortie de log plus propre.
//// --no-suggest: Évite d'afficher les suggestions de paquets supplémentaires.

```
    - name: Run PHPUnit tests
      run: ./vendor/bin/phpunit
```
//// Cette étape exécute les tests unitaires en utilisant PHPUnit. La commande ./vendor/bin/phpunit lance PHPUnit installé via Composer, qui exécute tous les tests définis dans le projet.
