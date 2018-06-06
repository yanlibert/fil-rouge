Après le déploiement du Jenkins Master, on doit paramétrer le plugin Kubernetes pour que Jenkins puisse se mettre à l'échelle tout seul. Pour cela, Jenkins doit avoir un rôle RBAC que l'on défini ici :

Pour ajouter dans le cluster un compte jenkins avec les droits admin sur les pods

```sh
kubectl create -f jenkins-service-account.yaml
```

Pour lui associer un secret : 
```sh
kubectl create -f build-jenkins-secret.yaml
```

Pour voir le token a mettre dans la config du plugin Kubernetes de Jenkins 

```sh
kubectl describe secrets/build-jenkins-secret
```
