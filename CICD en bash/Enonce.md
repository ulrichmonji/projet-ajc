# CICD en BASH

## Vue d'ensemble
Le concept de **CICD (Intégration et déploiement en continue)** est permet aux équipes IT d'avoir rapidement des résultats sur les changements éffectués sur l'application. il facilite aussi les mises en production de nouvelles versions applicatives.

Ce projet consiste à mettre en place une chaîne d'intégration à l'aide du **BASH**.

## Travail à faire
Vous disposez d'un site web statique disponible [ici](https://github.com/diranetafen/static-website-example). Votre travail est de conteneuriser cette application afin de la proposer sous forme d'image **docker**. Par là suite, cette image sera soumise à une chaîne CICD comportant les étapes suivantes : 
- *Etape 1* : **Lintage du Dockerfile**
- *Etape 2* : **Build de l'image**
- *Etape 3* : **Test de l'appication**
- *Etape 4* : **Scan de l'image**
- *Etape 5* : **Release de l'image**
- *Etape 6* : **Déploiement**

### Etape 1 : Lintage du Dockerfile
Pour le lintage, il faudrait utiliser un linter déja embarqué dans une image docker afin de vous simplifier la tâche. 

### Etape 2 : Build de l'image
Vous allez lancer le build de l'image docker et vous assurer qu'il se termine correctement.

### Etape 3 : Test de l'appication
Vous démarrez un conteneur de votre application et vous vérifiez qu'il correspond à vos attentes. Une fois le conteneur testé, ***il peut être supprimé*** pour ne pas saturer le serveur.

### Etape 4 : Scan de l'image
Vous allez scanner votre image afin de détecter des vulnérabilités. Vous avez le choux sur lle scanner à utiliser. **snyk** ou **trivy** ou toute autre solution, ce qui importe est le rapport de scan.

### Etape 5 : Release de l'image
Vous allez uploader l'image sur un registre. Vous avez le choix sur la solution de registre. **dockerhub** ou **registry + portus** ou **harbor**  ou toute autre solution, ce qui importe est de livrer l'image.

### Etape 6 : Déploiement
Etant donné deux environnements : **pré-production** et **production**, vous allez déployer votre application sur chacun de ces serveurs.

Pour des besoins de facilité, le déploiement se fera sur le cloud [heroku](https://devcenter.heroku.com/). Il vous faudra donc créer un compte sur heroku, et récupérer votre token d'accès personnal. Vous aller déclarer une variable d'environnement **HEROKU_API_KEY** qui va contenir ce token d'accès. 

Pour lancer les déploiements sur heroku, vous avez besoin d'installer son binaire. Pour des besoins de facilité, vous pouvez utiliser une image docker contenant déja le binaire heroku, il vous suffira juste de monter votre répertoire de travail dans ce conteneur. 

[Voici](rcm7/heroku-cli) un exemple d'image, qu'on pourra lancer comme ceci :
    
    docker run -dit --name heroku -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker  rcm7/heroku-cli

#### Example de déploiement

Soit les considérations suivantes :
- La variable d'environnement **HEROKU_API_KEY** est bien configurée
- le binaire **heroku** est installé

Etant donné l'envirronement de **production** et votre id de compte nommé **myID** par exemple, les lignes de commandes suivantes permettent de déployer l'application (que l'on baptisera **web**) dans Heroku: 

    heroku container:login
    heroku create myID-production || echo "project already exist"
    heroku container:push -a myID-production web
    heroku container:release -a myID-production web

Une fois le déploiement terminé, votre application sera disponible à une adresse sous cette forme : *https://**myID-production**.herokuapp.com*

