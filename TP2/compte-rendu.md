## service VScode lancé en mode Admin
## création du deployment.yaml :
- j'ai utilisé le template du cours, en changeant uniquement les noms et l'image créée au TP1 (question sur le fait que l'on écrive pas l'adresse complète)
- j'ai choisi de mettre le même nom partout, mais j'ai compris que le plus important c'est que le nom soit le même dans le label et le match label
- les noms sont transparents, et le cours était clair, je n'ai pas eu de problème sur les commentaires
## Déploiement : kubectl apply -f deployment.yaml
## trouver un moyen de tester
- j'ai compris la démonstration : kubectl describe pod pour obtenir l'IP d'un pod précis, puis un curl sur cette IP
- utilisation de la commande kubectl get pods pour avoir la liste des pods
