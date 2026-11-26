## Exercice 1 : Prise en main de Helm et CloudNativePG

**Installation**
- *helm version* : Helm disponible sur le VS Code d'Onyxia
- *helm repo add cnpg https://cloudnative-pg.github.io/charts* puis *helm repo update* : ajout du dépôt CNPG
- *helm search repo cnpg* : 5 charts, dont 2 utiles au TP
    - *cnpg/cloudnative-pg* : l'opérateur, déjà installé sur le SSP Cloud
    - *cnpg/cluster* (version 0.9.0) : un cluster PostgreSQL géré par l'opérateur

**Pattern opérateur** : CNPG ajoute un type d'objet *Cluster* (CRD) ; on décrit la base voulue, l'opérateur crée et gère les Pods, volumes, Services et Secrets

**Valeurs importantes de cnpg/cluster** (*helm show values cnpg/cluster*) indiquées par Claude
- *cluster.instances* : 3 par défaut → 1 dans le TP (pas de haute disponibilité)
- *cluster.storage.size* : 8Gi par défaut → 1Gi
- *cluster.initdb* : base *app* par défaut → base *reviews*, propriétaire *admin*
- Le reste (sauvegardes, restauration, réplication, supervision) n'est pas utilisé

## Exercice 2 : Un chart pour le backend

### 2.1 Squelette
- *helm create backend-app* : génération du chart
- Suppression des templates inutiles et conservation de *deployment.yaml*, *service.yaml*, *_helpers.tpl*, *NOTES.txt*

### 2.2 Développement
- *Chart.yaml* : *version: 0.1.0* (celle du chart), *appVersion: "1.1"* (celle de l'application, utilisée comme tag par défaut)
- *values.yaml* :
    - modifié : image *hugosmn5/backend-3-tier*, port 5000, probes *tcpSocket* comme TP5 
    - ajouté : *appName: backend* (nom du composant, pour réutiliser le chart), *containerPort*, *env* (*DB: reviews*)
    - supprimé : sections *ingress*, *serviceAccount*, *httpRoute*
- *deployment.yaml* :
    - nom : *{{ include "backend-app.fullname" . }}-{{ .Values.appName }}*
    - suppression de la ligne *serviceAccountName*
    - nom du conteneur : *{{ .Values.appName }}*
    - *containerPort* lu dans *.Values.containerPort*
    - ajout d'un bloc *env*
- *service.yaml* : nom identique au Deployment ; *targetPort: http* 
- *NOTES.txt* : remplacé par un message simple (nom du Service, commande pour les logs, fourni par Claude)

### 2.3 Validation et déploiement
- *helm lint ./backend-app* → *0 chart(s) failed*
- *helm template ma-release ./backend-app* : manifests générés conformes

![helm template](captures/Capture_tp7(1).png)

- *helm install ma-release ./backend-app* : release installée (REVISION 1), affichage du NOTES.txt
- *kubectl get deployments,svc,pods* : Deployment et Service créés ; Pod en *CrashLoopBackOff*
- *kubectl logs* : échec de connexion à PostgreSQL, attendu

![helm install et logs](captures/Capture_tp7(2).png)

### Difficultés rencontrées
- Très fastidieux de chercher les lignes à modifier
- *helm lint* : *NOTES.txt* utilisait des sections supprimées de *values.yaml* simplifié par Claude
- *helm lint* : *mapping values are not allowed* dans *deployment.yaml* : faute de frappe dans le template, corrigée
- Probes par défaut sur */* : remplacées par des probes TCP (vues en cours) pas pensé au premier coup, demandé à Claude.