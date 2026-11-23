# Debug Lab 1

## Déploiement
- Manifest de l'énoncé enregistré dans *debug-lab-service.yaml*
- *kubectl apply -f debug-lab-service.yaml*
- *kubectl get pods -l app=mon-api* donne bien 4 Pods en *Running* (2 alpha, 2 beta)

![délpoiement ok](captures/debug1(1).png)

## Constat
- Boucle de 10 requêtes vers le Service depuis le namespace (demandé à Claude) :
  *for i in $(seq 1 10); do curl -s -m 2 http://api-service || echo "ECHEC"; done*
- Résultat : des réponses *Reponse du deploiement Alpha* , mais des *ECHEC* pour Beta

![Boucle avant correction](captures/debug1(2).png)

## Diagnostic
- Les Pods Beta ne répondent jamais
- Comparaison des ports :
    - Alpha : application lancée avec *-listen=:5678*
    - Beta : application lancée avec *-listen=:8080*
    - *containerPort* : 5678 pour les deux
    - Service : *targetPort: 5678* pour tous les Pods
- Le Service sélectionne les 4 Pods (étiquette *app: mon-api*) et envoie le trafic sur le port 5678
- Les Pods Beta n'écoutent pas sur 5678

## Correction
- Dans *api-beta*, *-listen=:8080* remplacé par *-listen=:5678*
- *kubectl apply -f debug-lab-service.yaml*
- Nouvelle boucle de 10 requêtes : réponses Alpha et Beta alternées, aucun échec

![Boucle après correction](captures/debug1(3).png)

## Remaques 

- demandé à Claude la commande pour afficher uniquement les pods que je veux, kubectl *get pods -l app=mon-api*
- J'ai bien cru que la correction ne fonctionnait pas car, en relançant aussitôt, j'avais encore des echecs. Mais j'ai repensé que les pods ettaient du temps à être relancé, et c'était bien çà le problème.