# INFR250-M2RS

## Connection au cluster Openshift
Pour vous connecter au cluster Openshift mis à disposition :

### Pour trouver les sources rendez-vous dans la release suivante : https://github.com/okd-project/okd/releases/tag/4.19.0-okd-scos.19

### Pour Windows x86

Téléchargez sur votre machine le paquet OKD suivant : 
https://github.com/okd-project/okd/releases/download/4.19.0-okd-scos.19/openshift-client-windows-4.19.0-okd-scos.19.zip

Copiez le fichier oc.exe dans C:\Windows\System32

### Pour Linux x86
Téléchargez sur votre machine le paquet OKD suivant : 
https://github.com/okd-project/okd/releases/download/4.19.0-okd-scos.19/openshift-client-linux-4.19.0-okd-scos.19.tar.gz

Dézippez et copiez le fichier oc dans /urs/bin
```
tar -xvf openshift-client-linux-4.19.0-okd-scos.19.tar.gz
cp oc /usr/bin/
```

### Pour MAC

Télécharger le client suivant 

https://github.com/okd-project/okd/releases/download/4.19.0-okd-scos.19/openshift-install-mac-arm64-4.19.0-okd-scos.19.tar.gz pour ARM

https://github.com/okd-project/okd/releases/download/4.19.0-okd-scos.19/openshift-install-mac-4.19.0-okd-scos.19.tar.gz pour x86

Mettez le fichier oc dans le path.

Ou alors :
```
brew install openshift-cli
```


## Connexion au cluster (pour tous)

Allez sur l'url https://console-openshift-console.apps.openshift.kakor.ovh authentifiez-vous avec l'utilisateur ipi-gp-x (le x étant le numéro de groupe) en choisisant "KeystoneIDP".

Une fois authentifié (attention à ne pas recharger la page même si l'affichage prends du temps), allez en haut à droite de la page et sélectionnez l'option "Copy login Command" et réauthentifiez vous. 

Cliquez sur le lien Display Token et copiez dans votre terminal sur VScode la ligne de commande qui à été donné avec oc. 

Pour vérifier que vous êtes authentifiés, lancez la commande ```oc get pod```

## Exercice de remise en forme (avec livrable) 

Pour le dépot sur moncampus : 

- Les fichiers yaml de votre application et un README avec le nom du groupe. Dans un zip. (des commentaires dans le kustomization.yaml décrivant rapidement les actions du fichier). Une capture d'écran du site.

### Préparatif 

Créez sur la plateforme Openshift à disposition (ou la votre si vous avez) les éléments suivants

- Un deploiement avec un seul réplica avec l'image prise de prestashop/prestashop sur dockerhub nommé harbor.kakor.ovh/public/prestashop

Attention de mettre en place la variable d'en suivante pour éviter un crash loop :
```
        - name: PS_FOLDER_ADMIN
          value: admin-dev
```

- Un deploiement avec un seul réplica avec l'image prise de mariadb sur dockerhub nommé harbor.kakor.ovh/public/mariadb

- Liez les deux pod avec un secret dédié pour les mots de passe et les services qui vont bien 

- Enfin faites en sorte que l'application soit accessible depuis internet en <votreapp>.apps.openshift.kakor.ovh

- Pas de PVC 

Voici un exemple d'ingress :
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
    - host: nginx-correction.apps.openshift.kakor.ovh
      http:
        paths:
        - path: ''
          pathType: ImplementationSpecific
          backend:
            service:
              name: nginx
              port:
                number: 80
  tls:
  - {}
```

## Partie Kustomize 

Reproduisez cette arborescence à partir du travail précédent

```
├── someapp
    ├── base
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   └── service.yaml
    |   └── vosautrefichiers.yaml...
    └── overlays
        ├── dev
        │   ├── deployment.yaml...
        │   └── kustomization.yaml
        └── production
            ├── deployment.yaml...
            └── kustomization.yaml
```

En global, au déploiement existant on va ajouter quelques modifications via kustomize:

 - Faites en sorte que tous les objets possèdent un label avec vos noms
 - Le secret de mot de passe à la bdd doit être généré à partir d'un fichier plat (attention à la nomenclature)
 - faites en sorte que le pod de l'application prestashop possède une quantité de ram de 512 Mo max et 200m vpu max via un patch
 - Changez la version de l'image prestashop pour la mettre en version 9.0.1

Par environnements :

 - Ajoutez un préfixe à vos déploiements sous la forme "dev-" ou "prod-"
 - Pour les 2 déploiement de votre mariadb et prestashop, modifiez le nom de vos variables d'env database pour ajouter un -dev ou -prod