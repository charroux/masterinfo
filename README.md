# masterinfo

```
Put request : curl -X PUT -H "Content-Type: application/json" -d '{"begin" : "12/10/2023", "end" : "12/10/2023"}' "http://localhost:8080/cars/11AA22?rent=true"
```

### Docker

Build the Java file:
```
./gradlew build      
```

Check the Jar file in build/libs

Create a Dockerfile in the code folder: https://github.com/charroux/masterinfo/blob/main/rent/Dockerfile

Build a Docker image:
```
docker build -t rent .      
```
Run the container:
```
docker run -p 4000:8080 rent    
```
Then check in your browser:
```
http://localhost:4000/cars
```

### Publish the Docker image to the Docker Hub

Tager l'image :
```
docker tag 4da2332370c7 votreIdDocherHub/rent:1
```
où le numéro est l'identifiant de l'image donné par docker images, et 1 est un numéro de version

Se connecter au Docker Hub :
```
docker login
```

Publier l'image :
```
docker push votreIdDocherHub/rent:1      
```

### Launch a workflow when the code is updated

The script is there: https://github.com/charroux/masterinfo/blob/main/.github/workflows/actions.yml

Create a new branch:
```
git branch newcarservice
```
Move to the new branch:
```
git checkout newcarservice
```
Update the code and commit changes:
```
git commit -a -m "newcarservice"
```
Move to the main branch:
```
git checkout main
```
Push the changes to GitHub:
```
git push -u origin newcarservice
```
Create a Pull request on GitHub and follow the workflow.

Merge the branch on Github is everything is OK.

Then pull the new main version:

```
git checkout main
```
```
git pull origin main
```

If necessary delete the branch:

```
git branch -D newcarservice
```
```
git push origin --delete newcarservice
```

### Installer Minikube

https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download

### Démarrer Minikube
```
minikube start --driver=docker      
```

Combien de noeuds dans le cluster?
```
kubectl get nodes      
```

Dashboard pour inspecter le cluster :
```
minikube dashboard
```

Déployer votre image Docker :
```
kubectl create deployment rentalservice --image=charroux/rentalservice:1      
```
Attention d'utiliser votre image.

Vérifier que le procesus fonctionne bien :
```
kubectl get pods      
```

Ajouter un service :
```
kubectl expose deployment rentalservice --type=LoadBalancer --port=8080              
```    
Récupérer l'adresse du service :
```
minikube service rentalservice --url                      
```    
Tester dans votre navigateur :

http://127.0.0.1:50784/cars

En adaptant l'URL.

### Scalling

Récupérer l'adresse du service :
```
kubectl scale --replicas=2 deployment/rentalservice
```

Vérifier que vous avez bien 2 pods.

### Détruire un pod et constater qu'il redémarre automatiquement

Récupérer l'adresse du service :
```
kubectl delete pod (numéro du pod)
```

### Déployer via des fichiers yaml
```
kubectl apply -f deployment.yml           
```

Le fichier yaml : https://github.com/charroux/masterinfo/blob/main/deployment.yml

## Mettre en place une gateway

### Install Istio
https://istio.io/latest/docs/setup/getting-started/

Download Istio (take care at the version 1.17 here).

```
cd istio-1.17.0    
export PATH=$PWD/bin:$PATH    
istioctl install --set profile=demo -y
cd ..   
```
Enable auto-injection of the Istio side-cars when the pods are started:
```
kubectl label namespace default istio-injection=enabled
```
Install the Istio addons (Kiali, Prometheus, Jaeger, Grafana):
```
kubectl apply -f samples/addons
```
## 
Enable auto-injection of the Istio side-cars when the pods are started:
```
kubectl label namespace default istio-injection=enabled
```

Configure Docker so that it uses the Kubernetes cluster:
```
minikube docker-env
eval $(minikube -p minikube docker-env)
eval $(minikube docker-env)  
```

### Kubernetes Gateway

Check the configuration at 53: https://github.com/charroux/masterinfo/blob/main/deployment.yml

Check also where the Kubernetes service is registered in the gateway at line 72.

Apply again the config:
```
kubectl apply -f deployment.yml      
```

Then get the address of the gateway:
```
kubectl -n istio-system port-forward deployment/istio-ingressgateway 31380:8080  
```

and finally test in your browser:
http://localhost:31380/rentalservice/cars

### Security with Role-Based Access Control (RBAC)

Check the kinds Role and RoleBinding: https://github.com/charroux/masterinfo/blob/main/deployment.yml

#### Export the CA from Minikube (Minikube uses its own CA to manage the certificates
```
minikube ssh -- sudo cat /var/lib/minikube/certs/ca.crt > ca.crt
minikube ssh -- sudo cat /var/lib/minikube/certs/ca.key > ca.key
```

#### Generate a private key for test-user
```
openssl genrsa -out test-user.key 2048
```

#### Generate a certificate signature request (CSR)
```
openssl req -new -key test-user.key -out test-user.csr -subj "/CN=test-user"
```

#### Sign the certificate with the Minikube CA
```
openssl x509 -req -in test-user.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out test-user.crt -days 365
```

#### Add the user test-user to the Kubernetes configuration
```
kubectl config set-credentials test-user \
  --client-certificate=test-user.crt \
  --client-key=test-user.key
```

Create a context for test-user in the namespace rbac-test:
```
kubectl config set-context test-user-context \
--cluster=minikube \
--namespace=rbac-test \
--user=test-user
```

Change the context to test-user:
```
kubectl config use-context test-user-context
```

### Security tests
Lister the pods from the namespace rbac-lab 
```
kubectl get pods
```

Check that a new pod creation is forbidden:
```
kubectl run unauthorized-pod --image=nginx
```

### Clean-up
```
kubectl config use-context minikube
kubectl delete namespace rbac-test
kubectl config delete-context test-user-context
kubectl config unset users.test-user
```
