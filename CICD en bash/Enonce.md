# CICD en BASH

## Vue d'ensemble
Le concept de **CICD (Intégration et déploiement en continue)** est permet aux équipes IT d'avoir rapidement des résultats sur les changements éffectués sur l'application. il facilite aussi les mises en production de nouvelles versions applicatives.

Ce projet consiste à mettre en place une chaîne d'intégration à l'aide du **BASH**.

## Travail à faire
Vous disposez d'un site web statique disponible [ici](https://github.com/diranetafen/static-website-example). Votre travail est de conteneuriser cette application afin de la proposer sous forme d'image **docker**. Par là suite, cette image sera soumise à une chaîne CICD comportant les étapes suivantes : 
- **Lintage du Dockerfile**
- **Build de l'image**
- **Test de l'appication**
- **Scan de l'image**
- **Release de l'image**
- **Déploiement**

### Lintage du Dockerfile
Pour le lintage, il faudrait utiliser un linter déja embarqué dans une image docker afin de vous simplifier la tâche. 
### Build de l'image
### Test de l'appication
Vous démarrez un conteneur de votre application et vous vérifiez qu'il correspond à vos attentes. Une fois le conteneur testé, ***il peut être supprimé*** pour ne pas saturer le serveur.
### Scan de l'image
Vous allez scanner votre image afin de détecter des vulnérabilités. Vous avez le choux sur lle scanner à utiliser. **snyk** ou **trivy** ou toute autre solution, ce qui importe est le rapport de scan.
### Release de l'image
Vous allez uploader l'image sur un registre. Vous avez le choix sur la solution de registre. **dockerhub** ou **registry + portus** ou **harbor**  ou toute autre solution, ce qui importe est de livrer l'image.
### Déploiement
Etant donné deux environnements : **pré-production** et **production**, vous allez déployer votre application sur ces serveurs.
Pour des besoins de facilité, le déploiement se fera sur le cloud [heroku](https://devcenter.heroku.com/). Il vous faudra donc créer un compte sur heroku, et récupérer votre token d'accès personnalisé afin de pouvoir lancer les déploiements.Etant donné l'envirronement de **production** et votre id de compte **my_id** par exemple, les lignes de commandes suivantes permettent de déployer l'application (que l'on baptisera **web**) dans Heroku: 

    heroku container:login
    heroku create my_id-production || echo "project already exist"
    heroku container:push -a my_id-production web
    heroku container:release -a my_id-production web

Une fois le déploiement terminé, votre application sera disponible à une adresse sous cette forme : https://**my_id-production**.herokuapp.com