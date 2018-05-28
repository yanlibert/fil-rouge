### Presentation de fin de formation DevOps

Cette présentation est une application nodejs dont le  déployement sur notre CaaS Kubernetes va être entièrement automatisée selon la méthodologie DevOps.

## Creation du CaaS

Nous avons choisi Kubernetes comme support qui va nous permettre de déployer et gérer nos conteneurs de manière efficace.

Pour créer le cluster qui sera composé de 3 machines, nous pouvons :

- rester dans une optique classique et créer les instances sur les plateformes à la main puis installer les paquets nécessaires à Kubernetes

- honorer nos engagements d'automatisation dans l'esprit DevOps et utiliser des outils de création et d'approvisionnement des instances de support à Kubernetes.

Etant donné le nombre de serveurs impliqués dans notre projet, les deux approches sont valables et réalisables dans des temps d'implémentation similaires. Nous avons choisi la deuxième méthode qui, en plus de son intérêt pédagogique évident, nous permet une mise à l'échelle rapide et facilité.

# Creation des instances GCP et AWS

Nous allons utiliser l'outils de création et de gestion d'infrastructures Terraform. 