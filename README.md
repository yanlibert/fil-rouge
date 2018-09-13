# Petit POC DevOps

Le pinotage est un cépage Sud Africain très connu au Cap. Bien qu'il soit aussi ici question de grappe, nous en parlerons plutôt en terme de resources informatiques puisqu'un des sujets de ce projet est la création d'une "grappe de serveurs" en bon français. Plus communément appelé cluster, nous l'utiliserons comme plateforme de nos applications conteneurisées. 

Mais au delà du support et des considérations techniques, c'est véritablement une méthodologie orientée vers la culture DevOps que nous allons tenter d'entretenir tout au long de notre projet.

Dans cet esprit, c'est notre présentation de fin de projet qui va faire office de l'application principale a déployer. C'est une application nodejs dont le  déployement sur notre cluster fait maison va être entièrement automatisée selon la méthodologie DevOps. 

Sans compter la phase de préparation, ce projet est conçu pour être livré en une semaine par une équipe de trois adeptes DevOps en herbe. 

## Table des matières

   * [Petit POC DevOps](#petit-poc-devops)
      * [Table des matières](#table-des-matières)
      * [Les membres de l'équipe](#les-membres-de-léquipe)
      * [Sexy #DevOps](#sexy-devops)
      * [Phase de préparation](#phase-de-préparation)
         * [Postulat de départ](#postulat-de-départ)
         * [Quelques notions d'architecture](#quelques-notions-darchitecture)
         * [Le Workflow](#le-workflow)
      * [Creation des machines virtuelles](#creation-des-machines-virtuelles)
         * [Stockage de la box sur Vagrant Cloud](#stockage-de-la-box-sur-vagrant-cloud)
      * [Création du cluster Kubernetes](#création-du-cluster-kubernetes)
         * [Master](#master)
      * [Résolution d'un problème de réseau entre les nodes](#résolution-dun-problème-de-réseau-entre-les-nodes)
      * [Test du cluster](#test-du-cluster)
      * [Installation de Share Latex sur le cluster](#installation-de-share-latex-sur-le-cluster)
      * [Installation de Jenkins](#installation-de-jenkins)
         * [Pour accéder à Jenkins depuis l'extérieur](#pour-accéder-à-jenkins-depuis-lextérieur)

## Les membres de l'équipe

[Yannick Libert](https://github.com/yanlibert) @yanlibert

[Romain Delissen](https://github.com/Seiro10) @Seiro10

[Fabian Donini](https://github.com/FabDnn) @FabDnn

## Sexy \#DevOps

La méthodologie DevOps est très en vogue en ce moment. Tout le monde s'arrache "les DevOps" et bien des acteurs s'approprient ce terme. On parle de méthodologie, de philosophie, de culture, voir même de culte tout court dans certains [cercles](https://devops.com/devops-like-fitness-religion/)... De même qu'on ne peut pas se revendiquer "geek" parce qu'on aime regarder The Big Bang Theory, on n'est pas DevOps parce qu'on utilise Maven pour empaqueter une application Java. Et on n'est certainement pas "full DevOps" parce qu'on utilise Docker...

En revanche, il est vrai qu'une bonne maîtrise des outils favorise à entretenir le mythe. Quand un DevOps claque des doigts, des instances se créent dans le cloud, des cluster se forment, des applications se déploient et des mises à l'échelle s'implémentent toutes seules. 

<p align="center">
<img height="250" src="https://i.gifer.com/1R2R.gif">
</p>

Mais voilà : loin d'être des miracles, ces tâches ne peuvent réussir que si elles sont préparées avec soin selon une méthodologie standardisée très rigoureuse qui n'accepte que peu de compromis. C'est l'esprit DevOps, l'automatisation poussée à son paroxysme qui en théorie permettrait de developper une application sans coder une seule ligne. 

C'est d'ailleur ce que nous allons tenter dans ce projet. Notre objectif est de produire une application et son infrastructure qui la supporte sans générer une seule ligne de code dans le language natif de l'application. Dans ce cas précis, nous allons tenter de développer une application NodeJS sans écrire une seule ligne de Javascript.

*Accrochez vos ceintures, on passe en mode full DevOps*
<p align="center">
  <img height="350" src="./img/fulldevops.jpg">
</p>

## Phase de préparation

Pour accueillir notre application, nous avons besoin d'une plateforme. Et nous avons le choix ! Entre les IaaS, les PaaS, les CaaS, les SaaS et j'en passe, l'offre de services n'a jamais été aussi fournie. Pour faire notre choix, gardons à l'esprit que notre objectif est d'explorer les techniques d'automatisation en gardant une approche la plus agnostique possible et un contrôle complet des différentes couches logicielles (on parlera de stack par la suite). 

Pour ce projet, l'utilisation de conteneurs permet un déployment rapide et un gain de temps non négligeable sur toute la partie de mise en place et de configuration des applications. Par exemple, s'il nous faut un serveur Share Latex pour éditer notre rapport de projet, il suffira d'appeler un conteneur qui contiendra une image de Share Latex prête à l'emploi. 

Tout naturellement, pour gérer nos conteneurs nous utiliserons Docker. En plus d'offrir une solution stable, elle est largement supportée et documentée par une énorme communauté.

Au delà de la création de conteneurs, et puisque nous voulons automatiser au maximum les processus, pourquoi ne pas utiliser une solution qui va gérer nos conteneurs, les répartir sur nos machines selon leur ressources, surveiller leur santé et veiller à ce qu'ils soient toujours en fonction. 

> Parce qu'il ne faut pas oublier qu'une des qualités premières du DevOps est qu'il est très fainéant... 

Depuis peu, un acteur se détache de la guerre des orchestrateurs de conteneurs entre Kubernetes, Docker Swarm, OpenShift et d'autres. Kubernetes est supporté par une enorme communauté, offre une API solide et une grande flexibilité. L'envers de la médaille c'est qu'il faut un peu d'huile de coude pour tout mettre en place. 

Nous pourrions utiliser une solution tout-en-un (GKE ou AWS) mais le but de ce projet est d'apprendre. Alors apprenons !

### Postulat de départ

Nous avons à notre disposition 3 machines connectées à un réseau local. Nous pouvons :

- rester dans une optique classique et créer des machines virtuelles à la main puis installer les paquets nécessaires à Kubernetes

- honorer nos engagements d'automatisation dans l'esprit DevOps et utiliser des outils de création et d'approvisionnement des instances de support à Kubernetes.

Etant donné le nombre de nodes impliqués dans notre projet, les deux approches sont valables et réalisables dans des temps d'implémentation similaires. Nous avons choisi la deuxième méthode qui, en plus de son intérêt pédagogique évident, nous permet une mise à l'échelle rapide et facilité.

### Quelques notions d'architecture

Avant de se lancer tête baissée dans la création de notre cluster, il est nécessaire de bien en comprendre le fonctionnement. Cela nous aidera pour mettre en place les bonnes pratiques liées au déployement de nos applications. Nous allons nous baser sur une architecture standard de Kubernetes :

![Architecture fil-rouge](./img/archi.jpg)

- Serveur API : toutes les méthodes exposées par Kubernetes. l'API permet d'intéragir avec le cluster au moyen de JSON et de requêtes HTTP. Les requêtes mettent à jour l'état des objets stockés dans etcd. 

- Controller Manager : structure de contrôle qui régit l'état du cluster. Les requêtes API permettent d'indiquer au controlleur l'état désiré et le controlleur se charge d'y amener le cluster. 

- Scheduler : ou ordonnanceur en français permet de planifier les activités du cluster en fonction de la charge de travail, des performances et de la capacité du cluster. Il est assisté par les kubelets qui lui renseignent les performances du node sur lequel ils sont installés. 

- Etcd : la base de données qui stocke et gère les données de configuration du cluster. 

- kubectl : Kubernetes Command Line Tool. Permet d'utiliser l'API Kubernetes grâce à des commandes utilisables directement par les developpeurs.

- kubelet : gère l'état d'un noeud et prend en charge la gestion des conteneurs organisés dans les pods .

- Pod : un groupe d'un ou plusieurs conteneurs qui partagent des ressources de stockage et de réseau en commun. Les conteneurs au sein d'un même pod peuvent communiquer via localhost.

- Conteneur : contient une ou plusieurs applications avec tous les outils nécessaires pour les lancer. 

- kube-proxy : il fait office de proxy réseau et de répartiteur de charge. Il expose les ports nécessaire à la communication extérieure au cluster et route le traffic vers le conteneur approprié 

- Logging/ELK : la pile ElasticSearch, Logstash et Kibana qui va nous permettre d'avoir une solution d'analyse de log. 


### Le Workflow
> Git flow ou GitHub flow ? telle est la question...

Afin de limiter le chaos qu'engendre le développement collaboratif, il est crucial de décider d'un bon "flux de travail" ou workflow qui va définir la manière de gérer le tous les aspects de développement du projet.
Il existe plusieurs approches basées sur l'outil Git, le plus populaire étant Git flow.

Adapté pour la méthodologie agile, Git flow permet de gérer l'ajout de multiples fonctionnalités de manière indépendante sans intéragir directement avec la branche master. 

<p align="center">
  <img src="./img/gitflow.png">
</p>

*Un exemple de git flow*

Le git flow préconise l'utilisation d'une branche **master** et d'une branche **develop**. La branche master est la branche de production, elle porte les tags de version et les commit n'y sont pas fréquents. La branche develop est la branche des développeurs, elle sert de branche de base pour les features et les releases. S'ajoutent à ces branches de base des branches éphémères aux buts bien définis :
- **feature** pour ajouter une fonctionnalité
- **release** pour faire tampon entre la branche **develop** et la branche **master**
- **hotfix** pour gérer les bugs en urgence directement depuis le **master**

Si ces méthodes sont idéales pour une mise en production controlée et organisée, elles s'apparentent dans notre cas à l'utilisation d'un bazooka pour tuer une mouche. D'autant plus que cette gestion n'est pas idéale dans le cas de commit fréquents sur le master.

Nous allons donc utiliser une version beaucoup plus légère appelée **Github flow**. En plus d'être plus intuitive et d'offrir une courbe d'apprentissage bien plus confortable, elle permet de gérer les processus de développement directement dans le navigateur, grâce à la puissance de Github.

<p align="center">
  <img height="180" src="./img/githubflow.png">
</p>

Les étapes de Github flow s'articulent de cette manière :

- Création d'une branche depuis le repo
- Créer, éditer, supprimer des fichiers
- Envoyer une pull request  depuis la branche pour lancer une discussion entre dev
- Continuer les modification en mettant à jour la pull request
- Quand toutes les modifications sont prêtes, on les merge directement sur le master
- On supprime la branche
- Rinse and repeat

> Ou alors quand on est admin, on commit directement sur la branche master comme un cochon en faisant fi des toutes les règles (bon.. si c'est pour update le Readme on va pas m'en vouloir... Si ?)

Cette méthode va nous permettre des itérations rapides et des déploiement fréquents, ce qui sera vital étant donné le temps restreint alloué au projet.

Maintenant que les objectifs et l'organisation du travail sont définis nous allons pouvoir nous attaquer à la construction du cluster Kubernetes.

## Creation des machines virtuelles

Pour pouvoir créer des environnements de travail similaires à toute l'équipe, nous allons utiliser une image Ubuntu customisée avec Packer. 

Packer permet de créer et répliquer des machines virtuelles à l'aide de fichiers de configuration JSON. Pour créer les machines virtuelles, il suffit de lancer la commande :

```sh 
packer packer.json
```

Le fichier [packer.json](./packer/packer.json) permet de provisionner une VM avec le script [setup.sh](./packer/scripts/setup.sh) qui installe ```kubeadm``` et toutes ses dépendances

### Stockage de la box sur Vagrant Cloud

Notre box générée par Packer a été uploadée sur le Vagrant Cloud à l'adresse suivante : [https://app.vagrantup.com/dev2ops/boxes/kube](https://app.vagrantup.com/dev2ops/boxes/kube). De ce fait il est possible de la télécharger depuis n'importe où grâce aux fichiers Vagrantfile fournie dans le projet. Il suffit de copier le fichier dans le répertoire courant et de lancer :

```sh
vagrant up
```

Si la box n'a pas encore téléchargée, vagrant va la chercher sur Vagrant Cloud pour la stocker localement afin de l'utiliser plus rapidement la prochaine fois.

Nous avons donc 3 instances réutilisables à volonté si jamais le cluster ne fonctionne plus : 1 master et 2 nodes.

## Création du cluster Kubernetes 

### Master

En utilisant le [Vagrantfile](./vagrant/Vagrantfile-master) pour le master, on crée la VM en executant la commande :
```sh
vagrant up
```

> Remarque : par défaut la box utilise 1GB de mémoire RAM, ce qui est insuffisant pour le master. C'est pour cela qu'il y a une instruction de mémoire dans le Vagrantfile.

Pour se connecter à la machine, il suffit d'executer :

```sh
vagrant ssh
```

Une fois connecté sur le master, on peut initialiser le cluster : 
```sh
sudo kubeadm init
```
Si le cluster se lance sur l'ip interne au lieu de l'ip externe, relancer l'init avec la commande 
> Attention ! Ce n'est pas la bonne manière de traiter ce problème, cela ne fait que supprimer les syptômes d'un problème plus fondamental qui est traité [un peu plus bas.](#résolution-dun-problème-de-réseau-entre-les-nodes)

```sh
sudo kubeadm init --apiserver-advertise-address <ip de la machine>
```

Si le process s'interrompt, il est possible que la swap soit encore active sur le système. On peut la désactiver par la commande :

```sh
sudo swapoff -a
```

> Il faut le faire aussi sur les nodes si besoin. 

Ensuite on rajoute un réseau entre pod (qui ne fait pas partie de l'install de base de Kubernetes). Nous avons choisi Weave Net :

```sh
sudo sysctl net.bridge.bridge-nf-call-iptables=1
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
kubectl get pods --all-namespaces
```

A la fin de l'initialisation, executer les commandes indiquées et copier la commande ```kubeadm join``` pour joindre les noeuds au cluster. 

Sur les nodes, lancer la commande ```kubeadm join``` avec le token fourni à la création du cluster. Si il y a besoin de regénérer des token parce que celui d'origine a expiré, on peut en creer d'autre avec la commande : 

```sh
sudo kubeadm token create --print-join-command
```

Si quelque chose ne se passe pas bien et que l'on a besoin de supprimer un noeud :

```sh
kubectl drain <nodename> --delete-local-data --force --ignore-daemonsets
kubectl delete node <nodename>
```

Pour surveiller l'état du cluster, on peut utiliser la commande ```watch``` combinée aux commandes d'étât du cluser. Par exemple :

```sh
watch kubectl get nodes
```
et 
```sh
watch kubectl get pods --all-namespaces
```

Pour sauvegarder les deployments effectués sur le cluster :
```sh
kubectl get all --export=true -o yaml
```
Pour mettre le role node sur un node :

```sh
kubectl label node fabian node-role.kubernetes.io/node=
```
Pour voir les déployements sur le cluster :
```sh
kubectl get deployments
```

## Résolution d'un problème de réseau entre les nodes

Symptôme : tous les pods lancés sur le cluster restent en ```ContainerCreating``` et ne se lancent jamais. 

```kubectl describe pod <nom_du_pod>``` indique que le pod est bien alloué à un node mais le conteneur ne se lance pas. 

```journalctl -fu kubelet``` indique :

```txt
cni.go:171] Unable to update cni config: No networks found in /etc/cni/net.d
```

Solution : il se trouve que Vagrant crée une machine virtuelle avec deux interfaces eth0 pour le réseau local et eth1 pour le réseau externe qui porte l'adresse ip externe. 
On peut afficher l'interface par défaut en executant sur le master et les nodes la commande :
```sh
ip route list
```
Qui nous retourne :
```txt
default via 10.0.2.2 dev eth0 
```
Or ```kubeadm``` utilise l'interface par défaut du système pour créer le cluster. Il faut donc changer l'interface par défaut pour utiliser eth1

D'abord il faut connaître l'ip de la passerelle. Elle se trouve dans le fichier :

```sh
cat /var/lib/dhcp/dhclient.eth1.leases 
```
```txt
lease {
  interface "eth1";
  fixed-address 192.168.11.67;
  filename "\\Boot\\x64\\wdsnbp.com";
  server-name "nomduserver";
  option subnet-mask 255.255.255.0;
  option time-offset 7200;
  option routers 192.168.11.254;
  option dhcp-lease-time 604800;
  option dhcp-message-type 5;
  option domain-name-servers 172.16.0.15,8.8.8.8;
  option dhcp-server-identifier 192.168.11.254;
  option unknown-224 "FG100D3G14819332";
  option dhcp-renewal-time 302400;
  option dhcp-rebinding-time 529200;
  option netbios-name-servers 172.16.0.15;
  option domain-name "nomdudomain";
  renew 5 2018/06/08 22:29:44;
  rebind 2 2018/06/12 09:45:12;
  expire 3 2018/06/13 06:45:12;
}
```

Dans notre cas, l'ip de la passerelle est ```192.168.11.254```. Nous allons donc pouvoir reconfigurer l'interface par défaut : 

```sh
sudo ip route change to default dev eth1 via 192.168.11.254
```

On vérifie que la nouvelle interface par défaut est bien eth1 : 

```sh
ip route list
```

```txt
default via 192.168.11.254 dev eth1 
```

Merci à James Baltar qui indique comment changer d'interface par défaut [ici.](https://www.jamesbaltar.com/set-default-network-interface-in-ubuntu)

Il faut faire ce changement sur toutes les machines virtuelles de notre cluster. 

> TODO: ajouter ces modifications dans le packer.json

## Test du cluster

Pour lancer un déploiement test sur notre cluster, nous pouvons déployer hello-world avec cette commande :

```sh
kubectl run hello-world --replicas=1 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080
```
Puis vérifier que le conteneur se crée bien sur un des nodes :
```sh
watch kubectl get pods --all-namespaces
```
et

```sh
kubectl describe pod hello-world-5b446dd74b-l7kxd
```

Pour supprimer le déploiement on peut executer :

```sh
kubectl delete deployment hello-world 
```

## Installation de Share Latex sur le cluster

Share Latex va nous permettre de rédiger un rapport de projet de manière collaboratif.
Pour fonctionner, Share Latex a besoin de MongoDb (une base de données NOSQL) et de Redis (une base de données basée sur le principe clef => valeur).

Pour déployer ces dépendances, nous allons créer 3 fichiers de déploiement sur le Master (YAML) (liens vers les fichiers).
Les fichiers concernant le deploiement de Mongo et Redis auront deux fonctions principales :

1)Créer un service
Pour répondre à la problématique de réplication, nous devons regrouper nos pods via un service.
Ce service va agir comme un nom de domaine et sera capable de gérer le load balancing.
Si l'un des 2 noeuds tombe en panne , le second doit accueillir les pods de ce dernier.

2)Créer le déploiement
Cette étape consiste à configurer le pod en définissant notamment l'image qui sera utilisée, les ports nécessaires au bon fonctionnement de l'application, les variables d' environnement etc ...

La création de fichiers étant terminée, nous pouvons passer au déploiement en ligne de commandes.

Déploiement de Mongo : 

kubectl create -f deployment-Mongo.yaml

Déploiement de Redis : 

kubectl create -f deployment-Redis.yaml

Les dépendances installées, nous pouvons procéder au déploiement de Share Latex : 

kubectl create -f deployment-Sharelatex.yaml

Pour récupérer des informations sur l'état actuel des pods : 

kubectl get pods

Informations détaillées sur un pod : 

kubectl logs [nom du pod]

Share Latex est maintenant joignable et prêt à l'utilisation.

Quelques pistes pour le troobleshoting : 

Lorsque un soucis survient lors d'un déploiement, plusieurs commandes permettent d'essayer d'isoler la source de ce dernier.
Pour voir les logs : 

kubectl logs [nom du pod]

Concernant l'état actuel du déploiement, on peut utiliser : 

kubectl describe [nom du pod]

Lorsque vous souhaitez mettre à jour le déploiement, vous pouvez utiliser : 

kubectl apply -f [nom du fichier de deploiement] 

Si le deploiement n'existe pas, il sera crée. Si il existe il sera mis à jour.


## Installation de Jenkins

Nous allons nous baser sur le tutoriel disponible [ici](https://www.blazemeter.com/blog/how-to-setup-scalable-jenkins-on-top-of-a-kubernetes-cluster) que nous allons adapter pour ```kubeadm```.

L'image Jenkins que nous allons utiliser a été crée à l'aide du [Dockerfile](./jenkins/Dockerfile) et uploadée sur [dockerhub](https://hub.docker.com/r/fabiando/jenkin2). Il est donc facile de déployer Jenkins sur notre cluster à l'aide du fichier [jenkins-deployment.yaml](./jenkins/jenkins-deployment.yaml) fourni dans le repo.

On déploie Jenkins en executant :
```sh 
kubectl create -f jenkins-deployment.yaml
```

On crée un service s'appelant ```jenkins-service``` pour exposer le pod :
```sh
kubectl expose deployment jenkins --type=NodePort --name=jenkins-service
```
Pour afficher les services disponibles du cluster :
```sh
kubectl get service
```

### Pour accéder à Jenkins depuis l'extérieur

> Attention, ceci expose le serveur à l'extérieur du réseau local. Pour ce Proof Of Concept (POC), nous n'avons pas besoin de nous soucier de ce niveau de sécurité car toutes les machines sont en DHCP sur un LAN protégé. 

Puisque nos machines sont des machines Vagrant sans interface graphique, nous ne pouvons pas nous connecter sur localhost pour accéder à Jenkins. L'idée est donc d'utiliser nginx en mode reverse proxy. Le fichier de configuration nginx est disponible [ici](./nginx/jenkins). 

Après avoir modifier ce fichier en ayant renseigné l'ip locale du service exposé, il faut copier ce fichier dans le repertoire ```/etc/nginx/sites-available``` et créer un lien symbolique :

```sh
sudo ln -s /etc/nginx/sites-available /etc/nginx/sites-enabled
sudo systemctl reload nginx
```

> Rappel : pour afficher l'ip du service on execute la commande ```kubectl get services```
