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


## Partie HELM 

### Installation de Helm 

### Windows 

Téléchargez le fichier helm pour votre système d'exploitation 

https://get.helm.sh/helm-v3.21.0-windows-amd64.tar.gz

Récupérez dans l'archive le fichier helm.exe

copiez ce fichier dans le répertoire C:\Windows\System32

### Linux 

Exécutez les commandes suivantes :

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### MAC

Avec Brew, lancez la commande suivante :

```
brew install helm
```

## Reprise exercice Helm 

Installez la solution sur votre poste et connectez vous à Openshift comme indiqué dans l'étape 1 (utilisez le compte lié à votre numéro de salle)

Installez la chart helm https://artifacthub.io/packages/helm/cloudpirates-nginx/nginx

Lors de l'installation faites en sorte que l'ingress soit activé et utilise un hostname spécifique à votre groupe (groupe-2-nginx.apps.openshift.kakor.ovh) (si la route ne fonctionne pas ce n'est pas très grave)

Utilisez un fichier de values.yaml pour mettre en place ces options. 

Commande de correction à exécuter :

```
helm install nginx-correction oci://registry-1.docker.io/cloudpirates/nginx --version 0.12.0 -f helm/exonginx/values.yaml
```

## Exercice global helm : Variabilisation de prestashop. 

Le but va être dans un premier temps de créer une chart helm pour déployer l'application prestashop. 

Pour ce faire créez un nouveau projet helm nommé "prestashop" qui contiendra les sources de l'exercice kustomize sans la partie kustomize. 

Pour le moment je ne vous demande pas de variabiliser. Exécutez la commande helm template pour vérifier que votre application helm est bien déclarée. 

### Suite variabilisation

Dans votre chart, variabilisez :
- l'adresse de votre ingress
- le nom de votre deployment qui dépendra du nom de votre déploiement helm
- le nom des service prestashop et mariadb 

Testez que tout fonctionne en déployant dans votre namespace. Commencez à planifier l'arborescence de vos variables. 


### Fin variabilisation :

Via helm ajoutez la possibilité de pouvoir mettre en place ou non un ingress
Faites en sorte que le secret contienne une chaine de caractère générée automatiquement si un mot de passe n'est pas déclaré(via les fonction https://helm.sh/fr/docs/v3/chart_template_guide/function_list)

Dans le cas du deployment prestashop, faites en sorte avec un range que les valeurs d'environnement soient déclarées dans le fichier values (nom + valeur)

Mettez en place pour toutes les ressources des labels contenant une variable d'environnement et un nom

Générez un fichier NOTES.txt, qui permettra d'indiquer aux utlisateurs comment accéder à l'application si un ingress est déclaré. 