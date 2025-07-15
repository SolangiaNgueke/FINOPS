# FINOPS



1. Choix et préparation de l’application
Pour cet exercice, nous avons choisi une application open source disponible sur GitHub, développée par Brad Traversy, accessible à l’adresse : https://github.com/bradtraversy/mern-tutorial. Cette application est un bon candidat pour notre projet car elle correspond parfaitement aux critères demandés : elle possède un frontend, un backend et une base de données, sans fichiers Docker ni manifestes Kubernetes existants.
Présentation générale de l’application
L’application que nous avons choisie se nomme GoalSetter. Elle consiste à noter des “choses à faire”. Elle est une forme de todo-list et peut également servir de coffre-fort (conserver des mots de passe, conserver des tokens). Pour ce faire, il suffit de se connecter (ou de s’inscrire) avec une adresse mail.
<img width="920" height="348" alt="image" src="https://github.com/user-attachments/assets/375b8aff-0274-4a03-a756-f104bed75d8e" />






L’application est une application web full-stack basée sur la stack MERN, un acronyme pour MongoDB, Express.js, React et Node.js. Cette architecture est très populaire pour développer des applications web modernes et performantes. Le frontend est développé en React et backend est un serveur Node.js utilisant le framework Express.js, qui fournit une API REST pour gérer les requêtes du frontend, la logique métier, et l’interaction avec la base de données.
MongoDB est utilisé comme base de données NoSQL, adaptée à la gestion de données sous forme de documents JSON. Cette base permet une grande flexibilité pour stocker et manipuler les données liées aux utilisateurs, aux sessions, et à d’autres objets métiers de l’application.
Structure technique
Dans le dépôt GitHub, on trouve clairement séparément les dossiers pour chaque composant de l’application. Le dossier frontend/ contient tout le code source React, tandis que le backend est à la racine du projet avec le code serveur Node.js. MongoDB est utilisé via une image Docker officielle (dans notre futur docker-compose) mais n’a pas de code spécifique dans le dépôt : il s’agit d’une base de données externe que l’application utilise.

Composant
Technologie
Frontend
React + Redux
Backend
Node.js + Express
Base de données
MongoDB
API Authentification
JWT (JSON Web Token)
Déploiement
Docker + Docker Compose


Pourquoi ce choix ?
Ce projet remplit les critères clés du travail :
React est très utilisé en production, ce qui rend l’apprentissage applicable dans de nombreux contextes.


Base de données flexible : MongoDB s’adapte bien aux applications modernes et facilite la gestion des données.


Pas de dockerisation ni orchestration Kubernetes : l’absence de Dockerfile et Kubernetes permet de pratiquer la dockerisation et le déploiement Kubernetes de A à Z, ce qui correspond exactement à l’objectif du devoir.



2. Dockerisation des composants
Frontend 
L’architecture de notre frontend se présente comme suit: 
.
├── Dockerfile
├── README.md
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
└── src
    ├── App.js
    ├── App.test.js
    ├── app
    │   └── store.js
    ├── components
    │   ├── GoalForm.jsx
    │   ├── GoalItem.jsx
    │   ├── Header.jsx
    │   └── Spinner.jsx
    ├── features
    │   ├── auth
    │   │   ├── authService.js
    │   │   └── authSlice.js
    │   └── goals
    │       ├── goalService.js
    │       └── goalSlice.js
    ├── index.css
    ├── index.js
    ├── pages
    │   ├── Dashboard.jsx
    │   ├── Login.jsx
    │   └── Register.jsx
    ├── serviceWorker.js
    └── setupTests.js

Nous avons crée un fichier Dockerfile à l'intérieur de ce dossier et dans ce fichier, nous avons les informations suivantes: 
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Set environment variables for React in container
ENV CHOKIDAR_USEPOLLING=true
ENV WATCHPACK_POLLING=true

# Start the application
CMD ["npm", "start"]
 
Ce fichier dockerfile nous permet de créer un conteneur propre au Frontend qui sera par la suite utilisé dans le docker-compose.yml de notre application

Backend

L’architecture de notre backend se présente comme suit: 
.
├── Dockerfile
├── config
│   └── db.js
├── controllers
│   ├── goalController.js
│   └── userController.js
├── middleware
│   ├── authMiddleware.js
│   └── errorMiddleware.js
├── models
│   ├── goalModel.js
│   └── userModel.js
├── routes
│   ├── goalRoutes.js
│   └── userRoutes.js
└── server.js

Nous avons crée un Dockerfile à l’intérieur du serveur backend dont les informations sont les suivantes:
FROM node:18-alpine

WORKDIR /app

# Copy package files from root
COPY package*.json ./

# Install ALL dependencies (frontend + backend)
RUN npm install

# Copy entire project
COPY . .

# Change to backend directory for execution
WORKDIR /app

# Expose port
EXPOSE 5000

# Start the backend server
CMD ["npm", "run", "server"]
 

Création d’un docker-compose.yml
Nous n’avons pas créé d’image Docker pour Mongo car une image officielle existe déjà. On a juste spécifié Mongo dans notre fichier docker-compose.yml 
Notre code renseigné est: 
version: "3.8"

services:
  # MongoDB Database
  mongodb:
    image: mongo:6.0
    container_name: mern-mongodb
    restart: unless-stopped
    environment:
      MONGO_INITDB_DATABASE: mernapp
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    networks:
      - mern-network

  # Backend Service
  backend:
    build:
      context: .  # Contexte à la racine car package.json y est
      dockerfile: backend/Dockerfile
    container_name: mern-backend
    restart: unless-stopped
    environment:
      NODE_ENV: development
      PORT: 5000
      MONGO_URI: mongodb://mongodb:27017/mernapp
      JWT_SECRET: abc123
    ports:
      - "5000:5000"
    depends_on:
      - mongodb
    networks:
      - mern-network

  # Frontend Service
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: mern-frontend
    restart: unless-stopped
    environment:
      REACT_APP_API_URL: http://localhost:5000
      CHOKIDAR_USEPOLLING: true
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - mern-network

volumes:
  mongodb_data:

networks:
  mern-network:
    driver: bridge

Une fois ces fichiers créés, nous avons effectué un docker-compose –build  afin de créer des images docker.
L’image suivante montre que nos conteneurs ont bien été créé: 

Le frontend de l’application est à retrouver ici: http://localhost:3000
A présent, nous aller passer à la construction de manifests Kubernetes:

3. Construction de manifests Kubernetes
Pour rendre nos conteneurs plus scalables et surveiller leur état, nous allons passer à Kubernetes. Pour ce faire, nous allons créer à la racine un dossier k8s/ qui aura plusieurs dossiers : mongodb, backend, frontend, config/
L’arborescence de notre dossier k8s se présente comme suit: 

├── backend
│   ├── deployment.yaml
│   ├── secret.yaml
│   └── service.yaml
├── frontend
│   ├── deployment.yaml
│   └── service.yaml
└── mongodb
    ├── deployment.yaml
    ├── pvc.yaml
    └── service.yaml
Ensuite, on applique la commande  kubctl -apply -f frontend


4. Déploiement sur Minikube 
5. Intégration Kubecost
Après la création d’un compte sur notre application, des ressources kubernetes(pods et services) ont été provisionnées. Kubecost a détecté cette activité dans le cluster et a évalué les coûts associés.
Pour le moment, les coûts totaux sur les 07 derniers jours s’élèvent à 0,82$US.
Il n’y a actuellement aucune optimisation détectée (économies possibles = 0,00 $US/mois), et l’efficacité du cluster est de 0 %, ce qui indique probablement une utilisation très faible par rapport aux ressources allouées.


Dans l’image ci-dessous, on peut voir globalement les ressources allouées et utilisées, la répartition des coûts par Namespace qui est une séparation logique au sein d’un cluster Kubernetes.  
On constate que : 
Sur la barre CPU, l’Idle(gris) est énorme par rapport  à l’usage réelle(vert clair) par conséquent il y’a surprovisionnement de CPU 
Sur la RAM, il y’a une petite différence mais ne nécessite pas d’ajustement
le stockage n’a pas de problème de surprovisionnement 
Le namespace le plus utilisé est Kube-system.

Le dashboard ci dessous correspond aux coûts d’infrastructure Kubernetes, ventilés par type de ressource (Node, Disk, Network). On peut voir que le Node est la ressource la plus consommatrices










Après évaluation, nous avons constaté que les requêtes pour la création d’un compte  a un coût fixe soit 0,22$US ainsi nous pouvons avoir le tableau d’estimation de coût suivant : 


De plus, la majorité de ce coût est liée aux ressources réservées mais non utilisées (idle), qui représentent 1,57 $US soit plus de 83 % du coût total.
Les namespaces réellement actifs sont :
kube-system : 0,19 $US


kubecost : 0,13 $US


monitoring et default : négligeables (<0,01 $US chacun)


On observe que l'efficacité du cluster est de 0 %, ce qui signifie qu’aucune ressource réservée (CPU/RAM) n’est pleinement exploitée par les workloads déployés.




6. Surveillance avec Prometheus + Grafana 
Pour assurer un suivi en temps réel de l’état de notre cluster Kubernetes ainsi que des applications déployées, nous avons mis en place une solution de monitoring basée sur la stack Prometheus + Grafana.
Prometheus a été installé via Helm, scrutant tous les pods/nodes grâce au service discovery Kubernetes. Grafana, également installé via Helm, a été configuré pour utiliser Prometheus comme source de données, permettant la création de dashboards personnalisés.









Ce dashboard Grafana offre une vue synthétique de l’utilisation globale du CPU et de la RAM dans le cluster Kubernetes. On y compare la consommation réelle avec les ressources demandées (requests) et allouées (limits), ce qui permet de détecter rapidement un surprovisionnement ou un risque de saturation. Ce type de visualisation est essentiel pour optimiser les configurations et garantir une utilisation efficace des ressources du cluster.
Ce graphique montre l’évolution de l’utilisation CPU par pod sur les dernières minutes. Il permet de comparer facilement la consommation des différents pods et d’identifier ceux qui sollicitent le plus de ressources. Ce suivi détaillé est utile pour repérer des comportements anormaux ou optimiser la répartition des charges dans le cluster



Ce dashboard affiche l’utilisation du CPU et de la mémoire par namespace dans le cluster, ainsi que l’évolution de la taille du cache CoreDNS. Cette vue permet d’identifier rapidement les namespaces ou services les plus consommateurs en ressources, et de suivre le comportement du cache DNS, ce qui aide à anticiper d’éventuels problèmes de performance ou de saturation.


Ce graphique présente la limite CPU définie pour un pod spécifique (ici, un pod Kubecost). La courbe plate indique que la limite reste constante à 1,5 cœurs sur toute la période observée, ce qui permet de s’assurer que le pod ne dépassera jamais cette capacité CPU. Cette visualisation est utile pour vérifier que les limites de ressources sont correctement appliquées et respectées dans le cluster

7. 
Annexes

├── backend
│   ├── Dockerfile
│   ├── config
│   │   └── db.js
│   ├── controllers
│   │   ├── goalController.js
│   │   └── userController.js
│   ├── middleware
│   │   ├── authMiddleware.js
│   │   └── errorMiddleware.js
│   ├── models
│   │   ├── goalModel.js
│   │   └── userModel.js
│   ├── routes
│   │   ├── goalRoutes.js
│   │   └── userRoutes.js
│   └── server.js
├── docker-compose.yml
├── frontend
│   ├── Dockerfile
│   ├── README.md
│   ├── package-lock.json
│   ├── package.json
│   ├── public
│   │   ├── favicon.ico
│   │   ├── index.html
│   │   ├── logo192.png
│   │   ├── logo512.png
│   │   ├── manifest.json
│   │   └── robots.txt
│   └── src
│       ├── App.js
│       ├── App.test.js
│       ├── app
│       │   └── store.js
│       ├── components
│       │   ├── GoalForm.jsx
│       │   ├── GoalItem.jsx
│       │   ├── Header.jsx
│       │   └── Spinner.jsx
│       ├── features
│       │   ├── auth
│       │   │   ├── authService.js
│       │   │   └── authSlice.js
│       │   └── goals
│       │       ├── goalService.js
│       │       └── goalSlice.js
│       ├── index.css
│       ├── index.js
│       ├── pages
│       │   ├── Dashboard.jsx
│       │   ├── Login.jsx
│       │   └── Register.jsx
│       ├── serviceWorker.js
│       └── setupTests.js
├── package-lock.json
├── package.json
└── readme.md

Arborescence du projet


