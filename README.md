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
kubectl expose deployment rentalservice --type=LoadBalancer              
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