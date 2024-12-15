# kubernetes-devops-project

If you come from the Kubernetes training, you're in the right place ! :) 

Run the application :

docker-compose up -d


mettre a jour paquet apt-get update #installer gnome sudo apt install gnome-terminal #installer Docker sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

#Ajoutez la clé GPG de Docker : curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - #Ajoutez le dépôt Docker : sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" #Mettez à jour à nouveau la liste des paquets : sudo apt-get update #Installez Docker: Vous le pouvez maintenant installer Docker avec la commande suivante : sudo apt-get install docker-ce #Installer kubernetes curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
creer le projet dans service (srv)

cd /srv/projet-kubernetes
Créer les dossiers principaux

mkdir -p manifests/deployments mkdir -p manifests/services mkdir -p manifests/ingress mkdir -p configs/nginx mkdir -p configs/db mkdir scripts mkdir volumes
Créer un fichier README.md

touch README.md
Créer un fichier .gitignore

touch .gitignore

docker image prune #pour supprimer images non utilisees par un conteneur docker image prune

kubectl delete deployments --all -n default 
kubectl apply -f "./*.yaml"
 kubectl delete ingress --all -n default 
 kubectl delete svc --all -n default 
kubectl delete pvc --all -n default 
kubectl delete -f "./*.yaml"




#.1 Appliquer le Namespace kubectl apply -f YAML-STANDARD/namespace.yaml

#.2 Appliquer les Secrets kubectl apply -f YAML-STANDARD/secret-postgres.yaml #.3 Appliquer les Déploiements kubectl apply -f YAML-STANDARD/deployments/deployment-postgres.yaml kubectl apply -f YAML-STANDARD/deployments/deployment-pgadmin.yaml kubectl apply -f YAML-STANDARD/deployments/deployment-fastapi.yaml

#.4 Appliquer les Services kubectl apply -f YAML-STANDARD/services/service-postgres.yaml kubectl apply -f YAML-STANDARD/services/service-pgadmin.yaml kubectl apply -f YAML-STANDARD/services/service-fastapi.yaml# #.5 Appliquer l'Horizontal Pod Autoscaler (HPA) (si nécessaire) kubectl apply -f YAML-STANDARD/deployments/hpa-fastapi.yaml

#.6 Appliquer l'Ingress (si nécessaire) kubectl apply -f YAML-STANDARD/deployments/ingress-fastapi.yaml

#. Vérifier l'État des Ressources Après avoir appliqué toutes les configurations, vérifiez l'état des pods et des services pour vous assurer qu'ils fonctionnent correctement : kubectl get pods --namespace=standard kubectl get services --namespace=standard

    PostgreSQL Deployment

Rôle : Ce fichier définit le déploiement de la base de données PostgreSQL.

apiVersion: Définit la version de l'API Kubernetes utilisée pour ce déploiement (ici, apps/v1).
kind: Indique le type de ressource à créer (ici, Deployment).
metadata: Contient des informations sur le déploiement, comme son nom (postgres).
spec: Définit les spécifications du déploiement, notamment :
    replicas: Nombre d'instances de PostgreSQL à exécuter (ici, 1).
    selector: Labels utilisés pour sélectionner les pods gérés par ce déploiement.
    template: Modèle pour créer les pods, qui inclut :
        containers: Définit le conteneur PostgreSQL, y compris son image, les variables d'environnement (utilisateur, mot de passe, base de données, méthode d'authentification) et les ports exposés.
        volumeMounts: Monte un volume pour stocker les données de manière persistante.
volumes: Définit un volume nommé qui utilise un PersistentVolumeClaim (PVC) pour la persistance des données.

    PostgreSQL Service

Rôle : Ce fichier définit le service qui exposera PostgreSQL à d'autres applications dans le cluster.

apiVersion: Version de l'API Kubernetes pour le service (v1).
kind: Type de ressource (ici, Service).
metadata: Informations sur le service, comme son nom (postgres).
spec: Spécifications du service, y compris :
    ports: Définit le port sur lequel le service écoutera (5432).
    selector: Correspond à des pods ayant un label spécifique (ici, app: postgres) pour diriger le trafic vers le bon déploiement.

    FastAPI Deployment

Rôle : Ce fichier définit le déploiement de l'application FastAPI.

apiVersion: Version de l'API Kubernetes utilisée pour ce déploiement (ici, apps/v1).
kind: Type de ressource (ici, Deployment).
metadata: Contient des informations sur le déploiement, comme son nom (fastapi).
spec: Définit les spécifications du déploiement, notamment :
    replicas: Nombre d'instances de FastAPI à exécuter (ici, 1).
    selector: Labels utilisés pour sélectionner les pods gérés par ce déploiement.
    template: Modèle pour créer les pods, qui inclut :
        containers: Définit le conteneur FastAPI, y compris son image, les ports exposés, et la variable d'environnement DATABASE_URL pour se connecter à PostgreSQL.

    FastAPI Service

Rôle : Ce fichier définit le service qui exposera l'application FastAPI à l'extérieur du cluster ou à d'autres services internes.

apiVersion: Version de l'API Kubernetes pour le service (v1).
kind: Type de ressource (ici, Service).
metadata: Informations sur le service, comme son nom (fastapi).
spec: Spécifications du service, y compris :
    type: Type de service (LoadBalancer ou NodePort), qui détermine comment le service sera exposé.
    ports: Définit le port sur lequel le service écoutera (5000) et le port cible sur le pod (également 5000).
    selector: Correspond à des pods ayant un label spécifique (ici, app: fastapi) pour diriger le trafic vers le bon déploiement.

    PersistentVolumeClaim pour PostgreSQL

Rôle : Ce fichier définit une demande de volume persistant qui sera utilisé par PostgreSQL pour stocker ses données de manière durable.

apiVersion: Version de l'API Kubernetes utilisée pour le PVC (v1).
kind: Type de ressource (ici, PersistentVolumeClaim).
metadata: Informations sur le PVC, comme son nom (postgres-pvc).
spec: Spécifications du PVC, y compris :
    accessModes: Définit comment le volume peut être utilisé (ici, ReadWriteOnce, ce qui signifie que le volume peut être monté en lecture-écriture par un seul nœud).
    resources: Indique les ressources demandées, ici, une capacité de stockage de 1 GiB.

Pour déployer vos configurations Kubernetes, voici les étapes à suivre :

Vérifiez votre configuration de cluster : Assurez-vous que votre cluster Kubernetes est prêt et fonctionne. Vous pouvez vérifier en listant les nœuds du cluster :

kubectl get nodes

Créer les objets Kubernetes : Utilisez kubectl apply -f <chemin_du_fichier> pour appliquer vos fichiers de configuration. Vous pouvez appliquer chaque fichier individuellement ou bien appliquer tout un répertoire.

Par exemple, pour le rou d'eau, tous les fichiers sous le répertoiremanifests et le volume PostgreSQL :

kubectl apply -f /srv/projet-kubernetes/volumes/persistentVolumeClaim-postgreSQL.yaml kubectl apply -f /srv/projet-kubernetes/manifests/deployments/fastapi-deployment.yaml kubectl apply -f services/postgreSQL-service.yaml kubectl apply -f services/fastapi-service.yaml kubectl apply -f services/PGAdmin-service.yaml

érifier le statut des déploiements et services : Une fois les fichiers appliqués, vérifiez que les déploiements et services sont correctement créés.

kubectl get deployments kubectl get services

Vous pouvez aussi inspecter les pods pour vous assurer qu’ils sont bien créés et qu’ils n’ont pas d’erreurs :

kubectl get pods

#se placer dans app pour acceder au Dockerfile docker login -u julh62

#«Construire l'image docker build -t julh62/exam-kubernetes_fastapi:latest . #verifier image docker images

#Pousse docker push julh62/exam-kubernetes_fastapi:latest
si l'image existe deja

docker pull julh62/exam-kubernetes_fastapi:latest

#changer le nom de l'imge dans le fichier de deployment de fastapi containers:

    name: fastapi image: docker.io/library/fastapi-image:latest #appliquer les changeemnts et redeployer kubectl apply -f /srv/projet-kubernetes//manifests/deployments/fastapi-deployment.yaml

verifier connexion à la base de données

kubectl exec -it <nom_du_pod_postgresql> -- /bin/bash

#Tester la Connexion : Une fois à l'intérieur,
vous pouvez essayer de vous connecter à PostgreSQL avec le client psql :
psql -h localhost -U admin storedb

#recuper l'image pour pgadmin   (image officielle sur dockerhub)  
docker pull dpage/pgadmin4:latest


#augmenter ou diminuer le nbre de réplicas
nano hpa-fastapi.yaml
kubectl apply -f hpa-fastapi.yaml
kubectl get all

# verifier installation cert-manager*
kubectl get crds | grep cert-manager
#telecharger cert-manager
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.6.1/cert-manager.yaml

kubectl get all

nano clusterissuer-letsencrypt.yaml
kubectl apply -f clusterissuer-letsencrypt.yaml 
kubectl get clusterissuer letsencrypt-prod

#créer fichier 
nano ingress-fastapi.yaml
kubectl apply -f ingress-fastapi.yaml
#L'ingress est une ressource Kubernetes native comme les Pods, 
#les déploiements, etc. À l'aide de l'ingress, nous pouvons gérer les #configurations de routage DNS. L‘Ingress Controller effectue le 
#routage réel en lisant les règles de routage à partir des objets 
#d'entrée stockés dans etcd.
kubectl get all
kubectl get secret

kubectl logs -n cert-manager deploy/cert-manager
kubectl describe certificate fastapi-tls

kubectl get pods -n cert-manager



#sauvegarde base donnée maitre kubernetes
# a faire plus tard
k3s etcd-snapshot save 







#se connecter au conteneur postgres (cast-db) kubectl exec -it cast-db-dbbd5874c-6df89 -- bash find / -name "pg_hba.conf" vi /var/lib/postgresql/data/pg_hba.conf ajouter une ligne dans pg_hba.conf dans ipv4
Allow connections from all services in the Kubernetes cluster (in this case, from the 10.43.0.0/16 subnet).

host all all 10.43.0.0/16 trust

Résumé des touches importantes dans vi :

i : Passer en mode insertion (avant le curseur).
a : Passer en mode insertion (après le curseur).
Esc : Quitter le mode d'insertion et revenir en mode commande.
:wq : Sauvegarder et quitter.
:q! : Quitter sans sauvegarder.

#redemarrer le conteneur kubectl exec -it cast-db-dbyuhyh -- pg_ctl restart -D /var/lib/postgresql/data

http://jhennebo.be:"portduservice35444"/api/v1/movies/docs donne resultat not found
installer Ingress pour aider à la configuration des routes

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml kubectl delete -f movie-service-ingress.yaml kubectl delete -f cast-service-ingress.yaml kubectl delete ingress --all -n default kubectl get ingress
kubectl get ingressclass pour supprimer un ingress qui utilise traefik kubectl delete ingressclass traefik kubectl delete ingress movie-service-ingress -n default

kubectl get pods -n ingress-nginx kubectl apply -f movie-service-ingress.yaml kubectl apply -f cast-service-ingress.yaml

http://jhennebo.be/api/v1/movies/docs http://jhennebo.be/api/v1/casts/docs
Déploiement avec Helm
Installer Helm :

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

helm create fastapiapp Cela générera la structure nécessaire à une charte Helm dans le répertoire myhe>

fastapiapp/ ├── Chart.yaml # Métadonnées sur la charte (nom, version, description) ├── values.yaml # Valeurs par défaut pour les paramètres de la charte ├── templates/ # Fichiers YAML pour vos ressources Kubernetes │ ├── deployment.yaml │ ├── service.yaml

Déployer sur Kubernetes :

helm install fastapiapp ./fastapiapp

Mettre à jour le déploiement si nécessaire :

helm upgrade fastapiapp ./fastapiapp

Supprimer le déploiement :

helm uninstall fastapiapp

ne pas déplacer kubeconfig.yaml

Laissez kubeconfig.yaml à la racine de votre projet, où il est actuellement. Vous pouvez configurer Helm pour utiliser ce fichier si nécessaire, par exemple :

Ensuite, toutes vos commandes helm et kubectl utiliseront ce fichier. Option 2 : Configuration par défaut

Si votre cluster est déjà configuré avec kubectl (par exemple, via le fichier par défaut ~/.kube/config), vous n'avez pas besoin d'un fichier kubeconfig.yaml séparé. Dans ce cas, vous pouvez simplement supprimer ce fichier si vous êtes sûr de ne plus en avoir besoin. Résumé

Ne déplacez pas kubeconfig.yaml dans le chart Helm.
Utilisez-le comme configuration externe pour accéder au cluster.
Configurez votre environnement avec export KUBECONFIG=/path/to/kubeconfig.yaml si nécessaire.

Vérification et déploiement

Déploiement
Testez le fichier avec :

helm lint .

Cela vérifiera la syntaxe, les placeholders, et les valeurs utilisées dans values.yaml.

Ce que helm lint vérifie :

La syntaxe des fichiers YAML dans les templates.
La validité des valeurs dans values.yaml.
La structure du chart Helm.
La cohérence des références entre les fichiers.

Résultat :

Si tout est correctement configuré, vous devriez obtenir un message de succès avec des recommandations (s'il y en a).
Si des erreurs sont détectées, Helm vous indiquera où elles se trouvent pour que vous puissiez les corriger.

root@ubuntu:/srv/jenkins-exam/fastapiapp# helm lint . ==> Linting . [INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed root@ubuntu:/srv/jenkins-exam/fastapiapp#

La sortie de la commande helm lint . indique que votre chart est valide :

1 chart(s) linted, 0 chart(s) failed

Une fois toutes les modifications effectuées, exécutez la commande helm template pour générer à nouveau les fichiers YAML complets et vérifiez que tout semble correct. Vous pouvez rediriger la sortie vers un fichier et l’examiner : fastapiapp: service: port: 8083 # Port externe exposé par le service targetPort: 8083 # Port interne vers lequel le service redirige (port du conteneur) containerPort: 8083 # Port du conteneur où l'application écoute dans values.yaml pour rediriger le helm si besoin modifier service.yaml - port: {{ .Values.fastapiapp.service.port }} # Port exposé par le service (externe) targetPort: {{ .Values.fastapiapp.service.targetPort }} # Port cible dans le conteneur (interne) modifier deployment.yaml ports:

    name: http containerPort: {{ .Values.fastapiapp.service.containerPort }} # Port interne du conteneur protocol: TCP

helm template fastapiapp . -f values.yaml > output.yaml pour verifier les yaml reellement crées helm list pour voir si il y a deja une version du depoiement de helm

Déployer sur Kubernetes :

helm install fastapiapp ./fastapiapp

Mettre à jour le déploiement si nécessaire :

helm upgrade fastapiapp ./fastapiapp

Supprimer le déploiement :

helm uninstall fastapiapp

helm install fastapiapp . -f values.yaml Obtenez le nom du pod qui a été déployé
 avec le label fastapiapp 
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=fastapiapp,app.kubernetes.io/instance=fastapiapp" -o jsonpath="{.items[0].metadata.name}") Obtenez le port du conteneur dans lequel l'application écoute : export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}") Obetneir nom du pod echo $POD_NAME Vérifier les ports du pod kubectl describe pod fastapiapp-58dcb59864-kg7gr


##  pour l'instant jhennebo.be:31900/docs


