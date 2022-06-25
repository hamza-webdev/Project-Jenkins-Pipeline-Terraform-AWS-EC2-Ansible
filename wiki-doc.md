Welcome to the Project-Jenkins-Pipeline-Terraform-AWS-EC2-Ansible wiki!

# Step 1

1.  On a un dossier commingsoon: dedans un simple template html avec css et js et point d'entree index.html.
2.  Creer un fichier ### Dockerfile : image httpd et copy le dossier commingsoon dedans

    ```
    FROM httpd:latest

    COPY ./comingsoon /usr/local/apache2/htdocs
    ```

3.  Donner un nom a cette image et execute :
    ` docker build -t website-commingsoon . `

    > Afficher tous les images dockers:

         ````
          docker images
         ````

    > Creer un container de cette image (-d : run en arrière plan) et choisir les ports.

        ````
         docker run -d -p 8001:80 website-commingsoon
        ````

    > Open navigateur url: localhost:8001 : on voit notre sit web commingsoon

4.  Creer un Registry docker ensuite push dans dockerhub

```
# tagger cette image avant de pusher
$> docker tag website-commingsoon hamzabedwi/website-commingsoon:0.0.1
# il faut se connecter sur dockerhub avec docker login  -- ensuite tapper password de compte dockerhub
$> docker push hamzabedwi/website-commingsoon:0.0.1

```

## Dockercompose

### file requirement.txt pour installer lib necessaire pour notre project

1. creer un file docker-compose.yml

```
version: '3'

services:
    product-service:
        build: ./product
        volumes:
            - ./product:/usr/src/app
        ports:
            - 5001:80
    website:
        image: php:apache
        volumes:
            - ./website:/var/www/html
        ports:
            - 5000:80
        depends_on:
            - product-service
```

2. Execute le docker-compose a la racine .

```
docker compose up -d --build
```

## Ansible

> La grande force d’Ansible est qu’il est facile à mettre en œuvre, car il est agent-less et ne nécessite qu’une connexion SSH et la présence de python pour lancer les taches à réaliser.

Lorsqu’il y a plusieurs machines à gérer, Ansible exécute les opérations en parallèle. Cela permet de gagner un temps considérable. Cependant, les tâches sont effectuées dans un ordre définit par l’utilisateur lors du choix de la stratégie : Par défaut Ansible attendra d’avoir fini une tâche (sur tous les hôtes) pour passer à la suivante.

## Composants clés

- **Node Manage**r: ou control node, est le poste depuis lequel tout est exécuté via des connexions, essentiellement en SSH, aux nodes de l’inventaire.
- **Playbook**: Un playbook Ansible décrit une suite de tâches ou de rôles écrits dans un fichier ou format yaml.
- **Rôle**: Afin d’éviter d’écrire encore et encore les mêmes playbooks, les rôles Ansible apportent la possibilité de regrouper des fonctionnalités spécifiques dans ce qu’on appelle des rôles. Ils seront ensuite intégrés aux playbooks Ansible.
- **Inventory**: La liste des systèmes cibles gérés par Ansible est appelé un inventaire. On distingue deux types d’inventaire : l’inventaire statique constitué d’un fichier décrivant la hiérarchie des serveurs et l’inventaire dynamique fourni par un système centralisé recensant tous les nodes de l’infrastructure (ex NoCMDB)
- **Module**: Les tâches et les rôles font appel à des modules installés avec Ansible. Je vous invite à consulter la [liste sur le site d’Ansible](https://docs.ansible.com/ansible/latest/collections/index_module.html).
- **Template**: Comme son nom l’indique, un template est un modèle permettant de générer un fichier cible. Ansible utilise Jinja2, un gestionnaire de modèles écrit pour Python. Les « Templates » Jinja2 permettent de gérer des boucles, des tests logiques, des listes ou des variables.
- **Notifier**: indique que si une tâche change d’état (et uniquement si la tâche a engendré un changement), notify fait appel au handler associé pour exécuter une autre tâche.
- **Handler**: Tâche qui n’est appelée que dans l’éventualité où un notifier est invoqué
- **Tag**: Nom défini sur une ou plusieurs tâches qui peuvent être utilisé plus tard pour exécuter exclusivement cette ou ces tâches Ansible.

## Les principales commandes Ansible

> - **ansible**: Permet d'exécuter un simple module ansible sur un inventaire. Vu ci-dessus dans le premier test.

- **ansible-console** : Ouvre une console interactive permettant de lancer plusieurs actions sur un inventaire.
- **ansible-config** : Affiche l’ensemble des paramètres Ansible. ansible-config [list|dump|view].
  - list : affiche la liste complète des options d’Ansible à disposition.
  - dump : affiche la configuration dans le contexte actuel.
  - view : affiche le contenu d’un fichier de configuration Ansible
- **ansible-playbook** : Exécute un playbook Ansible sur un inventaire. La plus connue.
- **ansible-vault** : Permet de crypter des données qui ne doivent être divulgué.
- **ansible-inventory**: Affiche l’ensemble des données d’un inventaire Ansible.
- **ansible-galaxy** : permet d’installer des roles et des collections Ansible
- **ansible-doc** : Permet de lister l’ensemble des composants Ansible à disposition sur
  le noeud d’execution ansible-doc -l -t [type] type parmi: become, cache, callback,
  cliconf, connection,httpapi, inventory, lookup, netconf, shell, vars, module,
  strategy.

### Exemple de playbook.yml.

```
---
- name: "Build a container with ansible"
  hosts: localhost
  vars:
    version: 0.8.0
    version_precedent: 0.7.0
  connection: local
  tasks:
    - name: stop current running commingsoon-container
      command: docker stop commingsoon-container{{version_precedent}}
      ignore_errors: yes

    - name: remove stopped commingsoon-container
      command: docker rm commingsoon-container{{version_precedent}}
      ignore_errors: yes

    - name: remove website-comingsoon image
      command: docker rmi --force hamzabedwi/website-commingsoon{{version_precedent}}
      ignore_errors: yes

    - name: build docker image using the Dockerfile
      command: docker build -t hamzabedwi/website-commingsoon:{{version}} .

    - name: Push Image to dockerhub
      command: docker push hamzabedwi/website-commingsoon:{{version}}

    - name: run container **name= commingsoon-container
      command: docker run -d --name commingsoon-container{{version}} -p 5000:80 hamzabedwi/website-commingsoon:{{version}}
```

### Exécute le file de ansible-playbook.yml

```
ansible-playbook ansible-playbook.yml [--check]
```
