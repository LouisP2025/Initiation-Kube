# Première partie : Persistance de la base (StatefulSet)

- Suppression du Deployment *postgres* (le Service *postgres* est conservé)
- Création de *postgres-statefulset.yaml* : StatefulSet avec *volumeClaimTemplates* (1Gi), volume monté sur */var/lib/postgresql/data*
- Pod *postgres-0* en Running, PVC *data-postgres-0* en Bound
- Redémarrage du backend pour qu'il se connecte à la nouvelle base

- Avec le Deployment : lorsque l'on supprimait le Pod, erreur 500
- Avec le StatefulSet : après suppression de *postgres-0*, nouveau pod recréé avec le même nom et le même volume, l'avis est conservé

**Difficulté rencontrées**
- *postgres-0* en Error : *initdb: directory exists but is not empty* 
- Correction : variable *PGDATA* pointant vers un sous-dossier du volume, puis suppression du Pod en erreur pour qu'il soit recréé
- Je ne sais pas trop comment j'ai supprimé ses lignes dans une manip et je ne suis pas retourné voir le template du cours ensuite...