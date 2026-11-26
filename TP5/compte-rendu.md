# TP 5 : Mises à jour
## Exercice 1 : Probes

### Étape 1 : nouvelles images

- Route */health* ajoutée dans *TP1/python-api/src/main.py* (vérification renvoie *{"status": "ok"}*)
- Images construites et poussées sur Docker Hub : *1.0* : version initiale (*/hello*) », *2.0* : avec */health* et *3.0* : + */ensai*, préparée pour l'exercice 2
- Test local de chaque image : *docker run*, puis *curl http://localhost:8000/health* renvoie *{"status":"ok"}*
- Code envoyé sur GitHub, puis récupéré sur Onyxia

![Tags Docker Hub](captures/Capture_tp5(1).png)

### Étape 2 : probes 

- Copie du Deployment du TP 2 dans *TP5/python-api-deployment.yaml* avec l'image *lpellot/python-api:2.0*
- *readinessProbe* : *httpGet* sur */health*, port 8000, *initialDelaySeconds: 5*, *periodSeconds: 5*
- *livenessProbe* : *httpGet* sur */health*, port 8000, *initialDelaySeconds: 10*, *periodSeconds: 10*
- *kubectl apply -f python-api-deployment.yaml*
- Vérification avec *kubectl describe pod* :
    - lignes *Liveness* et *Readiness* conformes à la configuration
    - *Conditions* : *Ready True*
    - *Events* : aucun *Unhealthy*
- Vérification avec *kubectl logs* : appels réguliers *GET /health HTTP/1.1 200 OK*

![vérifications](captures/Capture_tp5(2).png)

### Étape 3 : tests d'erreurs probes

**Test A – readiness sur une route inexistante (*/jesaispas*)**
- Nouveau Pod *python-api-cdc949b7b-fccmk* en *Running* mais *0/1* : la readiness probe échoue, le Pod n'est jamais déclaré prêt
- Les 3 anciens Pods (*python-api-868c5f5857-...*) restent en *1/1* : Kubernetes ne les supprime pas tant que le nouveau n'est pas prêt
- La mise à jour est bloquée, mais l'API reste disponible grâce aux anciens Pods

![Test readiness](captures/Capture_tp5(3).png)

**Test B – même chose sur le liveness**
- *kubectl get pods* : les 3 Pods ont été remplacés par de nouveaux Pods en *Running 1/1*, avec les valeurs de *RESTARTS* qui augmentent
- La readiness (sur */health*) réussit : les nouveaux Pods sont déclarés prêts, la mise à jour va donc jusqu'au bout et les anciens Pods sont supprimés
- La liveness échoue : après 3 échecs consécutifs (environ 30 s), Kubernetes redémarre le conteneur, puis recommence, en espaçant les tentatives (*CrashLoopBackOff*)
- *kubectl get endpoints python-api* donne *not found*


- Différence avec le test A : la readiness bloque une mauvaise version, la liveness non ; elle redémarre seulement les conteneurs
- Retour à */health* sur les deux probes après chaque test.

**Pourquoi l'initialDelaySeconds de la liveness est-il plus grand ?**
- Une readiness trop tôt est sans gravité (le Pod n'est simplement pas encore prêt)
- Une liveness trop tôt ferait redémarrer en boucle un Pod qui n'a pas fini de démarrer

### Difficultés rencontrées

- *docker build* sans le point final encore une fois !
- *port is already allocated* : lors des tests sur mes images, un ancien conteneur occupait le port 8000. Solution Claude, *docker ps* puis *docker rm -f*
- comment tester les erreurs? Question au prof, regarder les logs et describe sur un pod.
- *One or more containers do not have resources - this can cause noisy neighbor issues* vsCode me prévient que je n'ai pas déclaré de ressources, mais cela ne pose pas de soucis pour l'exercice.

## Exercice 2 : RollingUpdate

### Mise en place
- Ajout d'une stratégie dans *TP5/python-api-deployment.yaml* :
    - *type: RollingUpdate*
    - *maxUnavailable: 1* : au plus 1 Pod indisponible pendant la mise à jour
    - *maxSurge: 1* : au plus 1 Pod en plus des 3 demandés
- Service *python-api* recréé (*kubectl apply -f TP3/service.yaml*) 

### Mise à jour vers la 3.0 (ajout de /ensai)
- Image remplacée par *lpellot/python-api:3.0* dans le manifest, puis *kubectl apply*
- Suivi avec *kubectl rollout status deployment/python-api* :
    - *2 out of 3 new replicas have been updated* : 2 nouveaux Pods créés d'un coup, Kubernetes attend qu'ils soient prêts
    - *1 old replicas are pending termination* : le dernier ancien Pod s'arrête proprement
    - *successfully rolled out* : les 3 Pods tournent en 3.0
- Vérification : *curl http://python-api/ensai* donne la réponse de la nouvelle route

![Rollout status et nouvelle route](captures/Capture_tp5(4).png)

### Conclusion
- Les anciens Pods ne sont supprimés qu'une fois les nouveaux déclarés prêts par la readiness probe
- Le Service n'est jamais modifié : seule l'image du Deployment change

### Essais des kubectl rollout

*kubectl rollout history deployment/python-api*
- 3 révisions affichées (2, 3, 4), la 4 étant la version 3.0 en cours
- La révision 1 n'apparaît plus : quand on réapplique une configuration identique à une ancienne révision, Kubernetes la renumérote au lieu d'en créer une nouvelle

*kubectl rollout undo deployment/python-api*
- *rolled back* : retour à la révision précédente,
- Avertissement : le Deployment avait été créé avec *kubectl apply* ; le rollback modifie le cluster mais pas l'annotation *last-applied-configuration*, donc le manifest ne correspond plus à ce qui tourne. Il est recommandé de revenir en arrière en réappliquant l'ancien fichier plutôt qu'avec *undo*

**Pause et reprise** : *kubectl rollout pause* puis *kubectl rollout resume*
- *paused* puis *resumed* : la pause suspend les mises à jour du Deployment ; aucune modification n'ayant été faite entre les deux, la reprise n'a déclenché aucun changement.
Il aurait été interessant de faire des modifications pour voir, mais je n'ai pas eu le temps pendant le TP.
- *kubectl rollout status* donne *successfully rolled out* : Deployment stable après la reprise

![Commandes rollout](captures/Capture_tp5(5).png)