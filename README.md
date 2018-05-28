# Presentation de fin de formation DevOps

Cette présentation est une application nodejs dont le  déployement sur notre CaaS Kubernetes va être entièrement automatisée selon la méthodologie DevOps.

## Creation du CaaS

Nous avons choisi Kubernetes comme support qui va nous permettre de déployer et gérer nos conteneurs de manière efficace. Nous pourrions utiliser une solution tout en un (GKE ou AWS) mais le but de ce TP est d'apprendre. Alors apprenons !

### Postulat de départ

Nous avons à notre disposition 3 providers qui nous fourniront les machines dont nous avons besoin.

- OVH : petite machine virtuelle payante. C'est celle qui as le plus de ressources. Elle soutiendra donc la machine maître (Master) de Kubernetes

- GCP : la PaaS de google. L'abonnement d'un an gratuit nous permet de creer une petite instance qui servira de noeud (node) Kubernetes

- AWS : la PaaS d'Amazon. Ici aussi l'instance virtuelle sera utilisé comme node.

Pour créer le cluster qui sera donc composé de 3 machines, nous pouvons :

- rester dans une optique classique et créer les instances sur les plateformes à la main puis installer les paquets nécessaires à Kubernetes

- honorer nos engagements d'automatisation dans l'esprit DevOps et utiliser des outils de création et d'approvisionnement des instances de support à Kubernetes.

Etant donné le nombre de serveurs impliqués dans notre projet, les deux approches sont valables et réalisables dans des temps d'implémentation similaires. Nous avons choisi la deuxième méthode qui, en plus de son intérêt pédagogique évident, nous permet une mise à l'échelle rapide et facilité.

### Creation des instances GCP et AWS

Nous allons utiliser l'outils de création et de gestion d'infrastructures Terraform. 