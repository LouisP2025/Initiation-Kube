#  Première partie : Exposition sur le réseau du cluster (Service)

- Deployment du TP 2 relancé (*tp2/deployment.yaml*) : 3 Pods *python-api*
- Création de *service.yaml* : Service de type *ClusterIP*
    - *selector: app=python-api* : cible les Pods du Deployment
    - *port: 80* et *targetPort: 8000* 
- *kubectl apply -f service.yaml*
- L'adresse réseau : *kubectl get service python-api* donne ClusterIP : *10.233.25.232*, port 80, pas d'adresse externe
- Test : *curl http://python-api/hello* répond bien World, c'est ok !

![Service python-api](captures/Capture_tp3(1).png)

**Difficultés rencontrées**
- Deployment du TP 2 supprimé du namespace : il a fallu le recupérer sur Git avant de créer le Service
- *the path "deployment.yaml" does not exist* : commande lancée depuis le mauvais dossier

## Deuxième partie : Exposition sur Internet (Ingress)

- Création de *ingress.yaml*, sur le modèle du cours (slide 37)
    - *host: python-api-lpellot.lab.sspcloud.fr* : adresse publique, qui doit finir par *.lab.sspcloud.fr* et être unique
    - *path: /* + *pathType: Prefix* : toutes les URL sont transmises
    - *backend* : Service *python-api*, port 80 (créé en partie 1)
- *kubectl apply -f ingress.yaml*
- Liste des Ingress : *kubectl get ingress*
- Test depuis le navigateur du poste de travail : copie de l'adresse dans le navigateur +/docs

![Liste des Ingress]!(captures/Capture_tp3(2).png)

![Test navigateur](captures/Capture_tp3(3).png)

**Difficultés rencontrées**
- Annotation *rewrite-target: /* de l'exemple du cours retirée, concerne une ancienne version.
- Colonne ADDRESS vide juste après la création, mais pas de soucis, c'était normal.

## Troisième partie : Application 3 tiers

## Partie 3 – Application 3 tiers

- *postgres.yaml* : Deployment *postgres:13* + Service *postgres* (5432)
    - Variables *POSTGRES_USER*, *POSTGRES_PASSWORD*, *POSTGRES_DB* : création de l'utilisateur et de la base *reviews* au démarrage
- *backend.yaml* : Deployment *hugosmn5/backend-3-tier:1.1* + Service *backend* (5000)
    - Variables *DB_USER*, *DB_PASS*, *DB*, *DB_HOST: "postgres"* (nom du Service de la base)
    - Nom de Service *backend* imposé : le frontend l'appelle par ce nom
- *frontend.yaml* : Deployment *hugosmn5/frontend-3-tier:1.1* + Service *frontend* (80 → 5000)
- *ingress.yaml* : expose uniquement le frontend sur *reviews-lpellot.lab.sspcloud.fr*
- Vérifications : *kubectl get pods*, *kubectl get services*
- Test : ajout d'un avis, puis affichage dans « Read Reviews »

![Avis enregistré](captures/Capture_tp3(4).png)

**Remarques**
- Seul le frontend est exposé ; le backend et la base restent internes
- Deployment et Service regroupés dans un même fichier, séparés par une ligne de trois tirets (demandé à Claude)

**Difficultés rencontrées**
- Ports du backend et du frontend non indiqués dans l'énoncé (demandé à Claude): trouvés dans les logs (*kubectl logs*), port 5000 pour les deux
- Attention à l'ordre de déploiement : base, backend, frontend. Il a fallu que je relance
- J'ai demandé à Claude pourquoi les valeurs des variables d'environnement sont entre guillemets : obligatoires si la valeur ressemble à un nombre ou à un booléen

