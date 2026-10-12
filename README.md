###  9. Personnaliser la page Web

### Différence entre un montage bind et la copie d'un fichier dans une image Docker

- **Montage bind** : le conteneur utilise directement le dossier de ma machine.
  Si je modifie le fichier `index.html`, le site change tout de suite.

- **Copie dans une image** : le fichier est copié dans l'image au moment de sa
  création. Il est figé : pour le modifier, il faut reconstruire l'image.

Dans ce TP, j'utilise un montage bind pour modifier ma page web facilement.

### 10. Créer un réseau Docker dédié

**Pourquoi le conteneur est-il connecté à ce réseau ?**
Le bloc  networks_advanced de main.tf le connecte à tp-iac-nginx-network.
Ce réseau est isolé : il sépare ce projet des autres conteneurs de la machine.

**Quel est l'intérêt de déclarer le réseau dans Terraform ?**
Le réseau est créé automatiquement avec le conteneur, de la même façon à chaque
fois, et il est supprimé automatiquement avec terraform destroy. Il n'y a
aucune commande manuelle à faire, et il est visible dans le code.

**Le réseau est-il supprimé à la destruction ?**
À vérifier à la fin du TP avec docker network ls après terraform destroy.


### 11. Limiter l'exposition du service

- `127.0.0.1` : le service est accessible uniquement depuis ma propre machine.
- Toutes les interfaces  : le service est accessible aussi depuis
  les autres machines du réseau.
J'utilise `127.0.0.1` pour ne pas exposer inutilement le site.


### 13. Inspecter l'état Terraform

**1. Quelles ressources apparaissent dans `terraform state list` ?**
Trois ressources : `docker_container.web` (le conteneur), `docker_image.nginx`
(l'image Nginx) et `docker_network.app_network` (le réseau Docker).

**2. Quelles informations affiche `terraform state show` ?**
Tout ce que Terraform sait du conteneur : son identifiant, son nom
(`tp-iac-nginx`), son image, ses ports (8080 vers 80), son montage (`html/`),
son réseau et son adresse IP (`172.19.0.2`).

**3. Pourquoi l'état Terraform doit-il être protégé ?**
Parce que c'est la mémoire de Terraform : il y note ce qu'il a créé. Si on le
modifie ou le supprime, Terraform ne reconnaît plus ses ressources. Il peut
alors en créer en double ou les perdre de vue. Il peut aussi contenir des
informations sensibles.

**4. Pourquoi éviter de le publier dans un dépôt public ?**
Parce que tout le monde pourrait le lire et y trouver des informations sur
l'infrastructure (identifiants, adresses, parfois des mots de passe). C'est
pour cela que `*.tfstate` est dans le fichier `.gitignore`.


### 15. Versionner avec Git

- **Revue** : `git diff` et l'historique montrent exactement ce qui a changé,
  par qui et pourquoi, donc on peut relire une modification avant de la valider.
- **Collaboration** : chacun travaille sur le même projet, et Git garde la trace
  des contributions de chacun.
- **Retour en arrière** : si une modification casse l'infrastructure, on peut
  revenir à une version précédente qui fonctionnait.


### 19. Exercices complémentaires
### Exercice 1 : changements prévus
Avec le nom `tp-iac-web`, Terraform prévoit : 2 ressources à créer et 2 à détruire.

- Le conteneur est supprimé puis recréé, car on ne peut pas renommer un conteneur.
- Le réseau est supprimé puis recréé, car son nom dépend de celui du conteneur.
- L'image Nginx ne change pas.

### Exercice 4 : dépendances

Le conteneur dépend de l'image, car il en a besoin pour démarrer, et du réseau, car il doit s'y connecter. Terraform crée donc l'image et le réseau d'abord, puis le conteneur.

### Exercice 5 : dérive de configuration

J'ai arrêté le conteneur à la main avec `docker stop tp-iac-nginx`. Quand j'ai lancé `terraform plan`, Terraform a vu que le conteneur ne tournait plus et a proposé de le recréer (1 à ajouter, 0 à modifier, 0 à détruire). J'ai fait `terraform apply` et le site marche à nouveau sur http://localhost:8082.

### Exercice 7 : nettoyage

On ne doit pas supprimer le fichier d'état à la main : Terraform ne saurait plus quelles ressources il gère, et le conteneur et le réseau resteraient sur la machine sans être suivis. Il faut utiliser `terraform destroy`.


### 20 Questions de compréhension

1. `terraform init` prépare le projet et télécharge le provider. `terraform plan` montre ce qui va changer. `terraform apply` applique les changements.

2. Un provider permet à Terraform de communiquer avec une technologie (ici Docker).

3. `terraform.tfstate` garde la correspondance entre le code et les ressources créées.

4. Les variables permettent de changer des valeurs sans modifier les ressources. Les outputs affichent des informations après le déploiement.

5. Une dérive est une différence entre le code et l'état réel de l'infrastructure.

6. L'IaC décrit l'infrastructure dans du code reproductible et versionnable. La configuration manuelle est faite à la main, avec plus d'erreurs et sans trace.

7. Git permet de suivre les modifications, relire, collaborer et revenir en arrière.

8. Un contrôle CI vérifie le code (formatage, validation). Un déploiement CD met en place automatiquement l'infrastructure ou l'application.

9. Il faut examiner le plan pour voir ce qui sera créé, modifié ou détruit, et éviter une mauvaise surprise.

10. Il ne faut pas publier l'état ni les fichiers de variables sensibles, et ne jamais écrire de secrets dans le code.