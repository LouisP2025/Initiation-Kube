# TP 5 : Mises à jour
## Exercice 1 : Probes

### Étape 1 : nouvelles images

- Route */health* ajoutée dans *TP1/python-api/src/main.py* (renvoie *{"status": "ok"}*)
- Images construites sur le poste de l'ENSAI (Docker absent d'Onyxia) et poussées sur Docker Hub :
    - *1.0* : version initiale (*/hello*) — tient lieu de « v1 »
    - *2.0* : + */health* — tient lieu de « v2 »
    - *3.0* : + */ensai*, préparée pour l'exercice 2
- Test local de chaque image : *docker run*, puis *curl http://localhost:8000/health* → *{"status":"ok"}*
- Code envoyé sur GitHub, puis récupéré sur Onyxia (*git pull*)

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
- Vérification avec *kubectl logs* : appels réguliers *GET /health HTTP/1.1 200 OK* (les probes de Kubernetes)

![vérifications](captures/Capture_tp5(2).png)

### Étape 3 : tests d'erreurs probes

**Test A – readiness sur une route inexistante (*/jesaispas*)**
- Nouveau Pod *python-api-cdc949b7b-fccmk* en *Running* mais *0/1* : la readiness probe échoue, le Pod n'est jamais déclaré prêt
- Les 3 anciens Pods (*python-api-868c5f5857-...*) restent en *1/1* : Kubernetes ne les supprime pas tant que le nouveau n'est pas prêt
- La mise à jour est bloquée, mais l'API reste disponible grâce aux anciens Pods

![Test readiness](captures/Capture_tp5(3).png)

**Test B – même chose sur le liveness**
- *kubectl get pods* : les 3 Pods ont été remplacés par de nouveaux Pods (*python-api-[...]*) en *Running 1/1*, avec une colonne *RESTARTS* à [N] qui augmente
- La readiness (sur */health*) réussit : les nouveaux Pods sont déclarés prêts, la mise à jour va donc jusqu'au bout et les anciens Pods sont supprimés
- La liveness échoue : après 3 échecs consécutifs (environ 30 s), Kubernetes redémarre le conteneur, puis recommence, en espaçant les tentatives (*CrashLoopBackOff*)
- *kubectl get endpoints python-api* donne *not found*


- Différence avec le test A : la readiness bloque une mauvaise version, la liveness non ; elle redémarre seulement les conteneurs

Retour à */health* sur les deux probes après chaque test.


### Difficultés rencontrées
- *docker build* sans le point final encore une fois !
- *port is already allocated* : lors des tests sur mes images, un ancien conteneur occupait le port 8000. Solution Claude, *docker ps* puis *docker rm -f*
- comment tester les erreurs? Question au prof, regarder les logs et describe sur un pod.