# Démonstration : Installation d'un Cluster Kubernetes as a Service chez Infomaniak

Ce dépôt contient les ressources et la documentation pour une vidéo de démonstration dédiée à l'installation et à la configuration d'un cluster Kubernetes as a Service chez Infomaniak. Cette démonstration couvre notamment la mise en place d'un contrôleur d'accès (Ingress) via _ingress-nginx_ et l'utilisation de _cert-manager_ pour la gestion des certificats SSL avec Let's Encrypt.

## Table des matières

- [Prérequis](#prérequis)
- [Déploiement de l'Ingress avec ingress-nginx](#déploiement-de-lingress-avec-ingress-nginx)
- [Installation de cert-manager](#installation-de-cert-manager)
- [Configuration de l'Issuer pour cert-manager](#configuration-de-lissuer-pour-cert-manager)
- [Références](#références)

## Prérequis

Avant de commencer, assurez-vous de disposer des éléments suivants :

- Un cluster Kubernetes opérationnel (Infomaniak offre un service de cluster Kubernetes)
- [Helm](https://helm.sh/) installé sur votre machine
- [Kubectl](https://kubernetes.io/docs/tasks/tools/) pour interagir avec le cluster

## Déploiement de l'Ingress avec ingress-nginx

Pour mettre en place un contrôleur d'accès via _ingress-nginx_, procédez comme suit :

1. **Ajouter le dépôt Helm d'ingress-nginx**  
   ```bash
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
   ```

2. **Installer ingress-nginx**  
   ```bash
   helm install ingress-nginx ingress-nginx/ingress-nginx
   ```

3. **Vérifier l'installation**  
   Après le déploiement, récupérez l'IP externe du contrôleur :
   ```bash
   kubectl get svc ingress-nginx-controller
   ```
   Vous devriez voir une sortie similaire à :
   ```
    NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
    ingress-nginx-controller   LoadBalancer   10.106.26.254   37.156.43.25   80:32446/TCP,443:30644/TCP   32h
   ```
4. **Zone DNS**
   Ajouter une zone DNS de type `A` dans le manager Infomaniak vers l'adresse IP affichée par la commande ci-dessus.


## Installation de cert-manager

Pour la gestion des certificats SSL (par exemple avec Let's Encrypt), installez _cert-manager_ en suivant ces étapes :

1. **Ajouter le dépôt Helm de jetstack**  
   ```bash
   helm repo add jetstack https://charts.jetstack.io
   helm repo update
   ```

2. **Installer cert-manager**  
   La commande suivante permet d'installer cert-manager tout en activant l'installation des CRDs (Custom Resource Definitions) nécessaires :
   ```bash
   kubectl create namespace cert-manager
   helm install cert-manager jetstack/cert-manager \
     --namespace cert-manager \
     --set crds.enabled=true \
     --set crds.keep=true
   ```
   *Note L'option `crds.keep=true` permet de conserver les CRDs en cas de désinstallation ou de mise à jour afin de ne pas perdre les définitions de ressources personnalisées.*

   Vous pouvez aussi exécuter l'installation en utilisant un fichier de valeurs personnalisé:
   ```bash
   helm install cert-manager jetstack/cert-manager \
     --values cert-manager/values.yaml \
     --namespace cert-manager
   ```

## Configuration de l'Issuer pour cert-manager

Une fois _cert-manager_ installé, vous devez définir un Issuer (ou ClusterIssuer) dans le même namespace que votre application (par exemple, un service FastAPI). Pour ce faire, appliquez le fichier de configuration:

Edité le fichier ``cert-manager/issuer.yaml`` avant pour remplacer le TODO

```bash
kubectl apply -f cert-manager/issuer.yaml
```

Ce fichier décrit le mode de fonctionnement de cert-manager en relation avec l'ACME (Let's Encrypt), permettant ainsi l'émission automatique des certificats.

## Déploiement de l'application demo_app

Pour déployer l'application, utilisez le fichier de configuration situé dans `demo_app/deployment.yaml`. Ce fichier contient la description complète des ressources nécessaires pour lancer l'application dans le cluster Kubernetes.

### Étapes de déploiement

1. **Vérifier la configuration**  
   Assurez-vous que les paramètres du fichier `demo_app/deployment.yaml` correspondent bien à votre environnement. Modifié le fichier pour remplir les TODO

2. **Appliquer le déploiement**  
   Exécutez la commande suivante pour déployer l'application dans votre cluster :
   ```bash
   kubectl apply -f demo_app/deployment.yaml
   ```

3. **Vérifier le déploiement**  
   Pour confirmer que le déploiement s'est déroulé correctement et que les pods de l'application sont en cours d'exécution, utilisez la commande suivante :
   ```bash
   kubectl get pods
   ```
   Vous devriez voir l'état des pods passer en `Running`.

## Références

Pour plus d'informations, consultez les ressources suivantes:
- [ingress-nginx sur Artifact Hub](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)
- [Documentation Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Guide Let's Encrypt - How It Works](https://letsencrypt.org/how-it-works/)
- [Getting Started avec Let's Encrypt](https://letsencrypt.org/getting-started/)
- [Cert-manager sur Artifact Hub](https://artifacthub.io/packages/helm/cert-manager/cert-manager)
- [Tutoriel cert-manager avec nginx-ingress](https://cert-manager.io/docs/tutorials/acme/nginx-ingress/)
- [Documentation d'utilisation du cert-manager avec Ingress](https://cert-manager.io/docs/usage/ingress/)