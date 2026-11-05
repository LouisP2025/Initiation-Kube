# Première partie 
## dezipper et inspecter le dossier
- repérer ce qui est dans le get, le fast api et uvicorn
## rédaction du Dockerfile :
- demande à Claude pour quoi mettre dans le FROM, je n'avais pas bien compris pendant le cours, maintenant c'est bon.
- oubli du . en fin du copy. Globalement, j'ai beaucoup de mal avec l'utilisation des "codes bash". Cela me ralenti beaucoup et me crée beaucoup d'erreurs.
- pour le reste, j'avais compris le principe, mais j'avais du mal à écrire la commande pour le pip install, j'ai demandé à Claude, même chose pour la ligne CMD
## Documentation des layers :
- faite au fur et à mesure,  vu qu'il fallait se poser la question avant de tout ce qu'il fallait indiquer.
## construction de l'image : docker build -t python-api:1.0 .
- problème de permission, contourné avec sudo, ensuite j'ai demandé à Claude pour ajouter mon compte au groupe docker.
- Je n'avais pas pensé au . Je ne trouvais pas comment indiquer le chemin, j'ai encore demandé à Claude
## exécuter : docker run -d -p 8000:8000 --name python-api python-api:1.0
- la ligne était dans le cours. Pendant la correction, j'ai essayé différentes options pour voir (j'ai demandé à Claude ce qu'était le mode détaché)
## Appel de l'API : curl http://localhost:8000/hello 
- un peu galéré pour la syntaxe de l'adresse
## Consulter les logs : docker logs python-api
## Pousser sur DockerHub :
- les commandes étaient entièrement dans le cours

# Deuxième partie
## dezipper et inspecter le dossier
- plus de mal (j'ai encore moins l'habitude de Java que de python). J'avais trouvé le GetMapping, mais j'ai donné le pom.xml à Claude pour les réglages.
## rédaction du Dockerfile :
- En demandant les explications du pom.xml, Claude m'a indiqué l'image à utiliser, 
- J'avais bien compris qu'il fallait mieux séparer en deux temps, mais je ne savais pas quoi mettre dans les deux commandes RUN
- la partie run était identique au cours
## Documentation des layers :
- faite au fur et à mesure, vu qu'il fallait se poser la question avant de tout ce qu'il fallait indiquer.
## construction, exécution, appel de l'API, logs et push :
- identique à la première partie avec java à la place de python.
## comparaison des tailles
- à l'aide de la commande docker images, la comparaison entre le image python et celle java n'a pas de sens. Il faudrait refaire une image construite en un seul stage en java pour comparer. Alors il est probable que celle en un bloc aurait un "content size" plus grand.
## .gitignore
-j'ai tenté après la correction. Je sais à quoi sela sert, mais je ne sais jamais quoi mettre dedans.



 
