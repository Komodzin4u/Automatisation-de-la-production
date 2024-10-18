# Automatisation-de-la-production

## TD Tests
### Introduction
L'intégration continue (CI) est une pratique de développement logiciel où le code est régulièrement testé et intégré dans un référentiel partagé, afin de détecter les erreurs plus tôt dans le cycle de développement. Ce TD a pour objectif de mettre en place un pipeline CI en utilisant GitHub Actions, pour exécuter automatiquement les tests unitaires via PHPUnit à chaque changement apporté à la branche principale (main).

Il installe le code source, configure l'environnement PHP, installe les dépendances nécessaires, puis exécute les tests pour s'assurer que le code fonctionne correctement.
L'objectif est d'avoir une intégration continue afin de détecter les erreurs de code rapidement après chaque modification.

### Explication du code
```
name: CI - PHPUnit Tests

on:
  push:
    branches:
      - main
```
On commence par spécifier les événements qui déclenchent ce workflow. Ici, le workflow s'exécute lorsque des modifications sont poussées vers la branche main.

```
jobs:
  test:
    runs-on: ubuntu-latest
```
runs-on: Spécifie l'environnement sur lequel ce job sera exécuté, ici ubuntu-latest, qui est une machine virtuelle avec le dernier Ubuntu disponible.
    
```
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
```
Cette étape utilise l'action actions/checkout@v3 pour récupérer le code du dépôt sur la machine virtuelle. C'est essentiel pour que les autres étapes aient accès aux fichiers du projet.

```
    - name: Set up PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.1' 
        extensions: mbstring, dom, curl, gd, sqlite3
        coverage: none
```
Cette étape configure PHP en utilisant l'action shivammathur/setup-php@v2.
php-version: Définit la version de PHP à utiliser, ici 8.1.
extensions: Installe les extensions PHP nécessaires (mbstring, dom, curl, gd, sqlite3).
coverage: Défini à none, cela signifie qu'aucun outil de couverture de code n'est configuré.

```
    - name: Install Composer dependencies
      run: composer install --prefer-dist --no-progress --no-suggest
```
Cette étape exécute la commande composer install pour installer les dépendances PHP du projet définies dans le fichier composer.json.
--prefer-dist: Préfère télécharger les archives des paquets (plus rapide que le clonage git).
--no-progress: Désactive l'affichage des informations de progression pour une sortie de log plus propre.
--no-suggest: Évite d'afficher les suggestions de paquets supplémentaires.

```
    - name: Run PHPUnit tests
      run: ./vendor/bin/phpunit
```
Cette étape exécute les tests unitaires en utilisant PHPUnit. La commande ./vendor/bin/phpunit lance PHPUnit installé via Composer, qui exécute tous les tests définis dans le projet.

### Conclusion
En résumé, ce workflow GitHub Actions permet d'automatiser les tests unitaires à chaque modification du code sur la branche principale, garantissant ainsi que les erreurs de code sont détectées rapidement. Cela améliore considérablement la qualité du projet et permet aux développeurs de travailler en toute confiance, en sachant que le code est testé automatiquement.


## TD Code Coverage
### Introduction

Dans ce TD, l'objectif est d'aller au-delà des simples tests unitaires en ajoutant la génération de rapports de couverture de code. La couverture de code est une mesure qui indique dans quelle proportion le code source est couvert par les tests. Ce processus va permettre d'assurer une bonne qualité du code, en identifiant les parties non testées et donc potentiellement vulnérables.

Dans ce TD, plusieurs tâches ont été réalisées afin d'obtenir le ci.yml décrit plus bas :

- Générer un rapport de couverture en texte localement avec PHPUnit.
- Tester différentes options de sortie pour les rapports de couverture.
- Générer des rapports de couverture de code dans GitHub Actions au format texte et Cobertura.
- Utiliser l'action GitHub irongut/CodeCoverageSummary pour obtenir un résumé de la couverture.

### Intégration du code coverage dans GitHub Actions
Nous avons intégré la génération de couverture de code, ainsi que la publication du rapport en format Cobertura. Voici les ajouts effectués dans le workflow :

On utilise le format Cobertura pour la couverture de code.
```
- name: Run PHPUnit tests with Cobertura coverage
  run: ./vendor/bin/phpunit --coverage-cobertura=coverage.xml
```

Le rapport est ensuite uploadé comme un artefact GitHub pour une consultation ultérieure :
```
- name: Upload Cobertura Coverage Report
  uses: actions/upload-artifact@v3
  with:
    name: cobertura-coverage
    path: coverage.xml
```

L'action irongut/CodeCoverageSummary a été utilisée pour ajouter un résumé de la couverture directement dans les commentaires du workflow sur GitHub. Cette action permet de générer un badge qui indique visuellement le pourcentage de couverture, et de définir des seuils minimaux de couverture.
```
- name: Code Coverage Report Summary
  uses: irongut/CodeCoverageSummary@v1.3.0
  with:
    filename: coverage.xml
    badge: true
    fail_below_min: true
    format: markdown
    hide_branch_rate: false
    hide_complexity: true
    indicators: true
    output: file
    thresholds: '60 80'
```

Enfin, nous affichons les résultats dans le résumé du job GitHub Action :
```
- name: Display coverage results
  run: |
    echo "## Code Coverage Results" >> $GITHUB_STEP_SUMMARY
    cat code-coverage-results.md >> $GITHUB_STEP_SUMMARY
```

### Conclusion
Grâce à ce workflow, nous avons non seulement automatisé l'exécution des tests, mais aussi la génération et l'affichage de rapports de couverture de code, facilitant ainsi le suivi de la qualité du code sur chaque push. L'intégration des badges et des résumés de couverture dans GitHub permet aux développeurs de surveiller la couverture du code en un coup d'œil et de prendre de meilleurs décisions pour améliorer la qualité du projet.

## TD Analyse Statique
### Introduction
Dans ce TD, l'objectif est d'améliorer la qualité du code en automatisant l'analyse statique avec des outils spécifiques à PHP : PHPCS, PHPMD et PHPStan. Ces outils permettent de détecter les problèmes potentiels dans le code avant même son exécution. En les intégrant à notre pipeline CI, nous garantissons une meilleure qualité de code tout au long du développement.

### Intégration de PHPCS, PHPMD et PHPStan dans le worflow
#### PHPCS
PHP CodeSniffer (PHPCS) est un outil qui analyse le code pour vérifier s'il respecte les conventions de codage définies. Il est particulièrement utile pour maintenir une cohérence dans le style de code entre différents développeurs d'une équipe.

Voici l'ajout de PHPCS dans le fichier CI :
```
- name: Run PHPCS
  run: ./vendor/bin/phpcs --extensions=php ./lib/
  continue-on-error: true
```
Cette commande analyse tous les fichiers PHP du répertoire lib/ et renvoie les erreurs de style.
L'option ``continue-on-error: true`` permet de ne pas bloquer le workflow si des erreurs de style sont détectées. Cela permet de remonter les problèmes tout en laissant le reste du pipeline continuer.

#### PHPMD
PHP Mess Detector (PHPMD) est un outil d'analyse qui repère les "mauvaises pratiques" dans le code, comme les fonctions trop complexes, les méthodes inutilisées ou les variables trop longues. L'objectif est de rendre le code plus lisible et maintenable.

L'ajout de PHPMD dans le workflow se fait de manière similaire à PHPCS :
```
- name: Run PHPMD
  run: ./vendor/bin/phpmd ./lib ansi phpmd.xml
  continue-on-error: true
```
Le format ansi permet d'obtenir des résultats en couleurs dans le terminal, ce qui rend l'analyse plus lisible.

#### PHPStan
PHPStan est un outil d'analyse statique plus avancé qui repère des erreurs potentielles dans le code, comme des erreurs de type, des variables non initialisées ou des appels de fonctions incorrects. Contrairement à PHPMD, il se concentre plus sur la logique du code que sur sa complexité.

PHPStan est intégré de manière similaire aux autres outils :
```
- name: Run PHPStan
  run: ./vendor/bin/phpstan analyse lib
  continue-on-error: true
```

### Conclusion
L'intégration de ces outils d'analyse statique dans notre workflow CI permet de détecter rapidement des erreurs de style, de structure et de logique, ce qui améliore la qualité du code et facilite sa maintenance. Corriger les erreurs remontées par PHPCS, PHPMD et PHPStan, permet non seulement d'avoir un code plus propre, mais aussi moins de risques de rencontrer des bugs en production.

## TD Déploiement continu
### Introduction
L'objectif de ce TD est d'automatiser le déploiement du site web sur un serveur FTP en utilisant GitHub Actions. Le déploiement FTP permet de transférer automatiquement les fichiers du projet vers un serveur de production à chaque mise à jour de la branche principale. Cela permet d'éviter les manipulations manuelles et garantit une mise à jour rapide du site après chaque modification.

### Créations des secrets GitHub
Avant de configurer le déploiement FTP, on a récupérer notre login et mot de passe pour le serveur FTP, ainsi que l'URL du serveur. Ces informations sont sensibles et doivent donc être stockées de manière sécurisée dans GitHub sous forme de secrets. Cela permet d'éviter de les exposer publiquement dans les fichiers du projet.

Nous avons ajouté trois secrets dans les paramètres du dépôt GitHub :
- SERVER : l'URL du serveur FTP
- ID : le nom d'utilisateur pour le serveur FTP
- CREDENTIALS : le mot de passe associé

Ces secrets sont ensuite utilisés dans le workflow GitHub Actions pour permettre l'accès sécurisé au serveur lors du déploiement.

### Déploiement via GitHub Actions
Pour automatiser le déploiement, nous avons ajouté une étape au workflow CI, qui utilise l'action FTP Deploy. Voici les ajouts effectués dans le fichier CI :
```
web-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy to FTP server
        uses: SamKirkland/FTP-Deploy-Action@4.3.1
        with:
          server: ${{ secrets.server }}
          username: ${{ secrets.id }}
          password: ${{ secrets.credentials }} 
          server-dir: www/
          exclude: |
            **/.git/**
            vendor/
```
- **Checkout code** : Cette étape récupère le code source depuis le dépôt GitHub pour qu'il soit disponible sur la machine virtuelle.
- **Deploy to FTP server** : L'action ``SamKirkland/FTP-Deploy-Action`` est utilisée pour transférer les fichiers vers le serveur FTP.
  - ``server`` : L'adresse IP ou l'URL du serveur FTP, récupérée des secrets GitHub.
  - ``username`` et ``password`` : Les identifiants du serveur FTP, également stockés en tant que secrets.
  - ``server-dir`` : Le répertoire cible sur le serveur, ici ``www/``, qui correspond à l'emplacement où les fichiers doivent être déployés.
  - ``exclude`` : Les fichiers ou dossiers à exclure du transfert, tels que ``.git/`` et ``vendor/`` pour éviter de déployer des fichiers non nécessaires.

### Conclusion
Ce workflow permet de simplifier le processus de déploiement en automatisant la mise à jour du site web à chaque push sur la branche principale. Grâce à l'utilisation des secrets GitHub, les informations sensibles sont protégées, et le site est déployé de manière sécurisée.
