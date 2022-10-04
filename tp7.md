# Compte-rendu TP7 Ansible

## Exercice 1 : Mise en place de l’architecture technique

### Question 1 : Préparation des machines

1.  Dans les paramètres de VirtualBox (menu Fichier > Paramètres > Réseau), on commence
par créer le réseau NAT correspondant : on crée un nouveau réseau NAT avec l'icone de droite, on fait modifier le réseau pour lui attribuer l'adresse IP du réseau indiqué : 192.168.122.0/24.

2. On va dans les paramètres de la VM et on lui rajoute une carte réseau que l'on configure en Réseau NAT.

3. On va attribuer à cette nouvelle carte réseau l'adresse IP renseignée pour le Node Manager : 192.168.122.10. Pour cela on va modifier le fichier Netplan avec la commande `sudo nano /etc/netplan/(ici on effectue une tabulation pour que le système trouve lui même le fichier en .yaml, étant donné qu'il n'y en a qu'un seul.)`.     On se met sous la ligne du dhcp, aligné avec la première carte réseau qui est l'interface en NAT, qui a donc accès à internet, on utilise que des retours à la ligne et des espaces. On inscrit le nom de la carte réseau enp0s8 et son adresse, ainsi :

enp0s8: addresses: - 192.168.122.10/24

On sauvegarde et on quitte, on fait ensuite un sudo netplan try puis un sudo netplan apply. On vérifie la configuration en faisant un ip a.

### Question 2 : Résolution des noms

Pour faire une résolution de noms sans pour autnant configurer tout un serveur DNS, on modifie le fichier /etc/hosts et on ajoute les deux entrées suivantes au fichier /etc/hosts :
`sudo nano /etc/hosts
- 192.168.122.11 http1
- 192.168.122.12 bdd1`

On peut alors cloner cette machine pour former deux autres machines, un serveur Web et un serveur de BDD. Avec VirtualBox, machine éteinte, on clone cette machine en faisant un clic droit sur la machine, cloner, on peut renommer la machine puis on confirme.

### Question 3 : Test de la configuration

1. On modifie les adresses IP des serveurs Web et serveurs BDD. Pour cela on va modifier les fichiers de configuration Netplan de chaque machine, comme efectué précédemment. On oublie pas de faire `sudo netplan try` puis entrer si aucune erreur n'est survenue ou directement `sudo netplan apply` pour appliquer directement les changements. On fait un `ip a`pour vérifier que l'adresse IP a été changée. De la même manière pour le serveur de BDD, avec l'adresse 192.168.122.12.

2. Pour changer les noms des machines on exécute la commande `hostnamectl hostname http1`. On va ensuite modifier le fichier /etc/hosts de manière à remplacer le nom 'server' par 'http1'. On enregistre et on quitte, on reboot et on vérifie en exécutant la commande `hostname`, la machine nous indique bien http1. On fait la même chose pour le serveur de BDD.

3. Pour que les trois machines puissent communiquer ensemble, leurs adresses MAC doivent être différentes. On génère donc des adresses aléatoires pour les deux cartes réseau des machines http1 et bdd1. On effectue un ping depuis la machine http1 sur la machine bdd1 : `ping bdd1`.
On essaye de ping Google pour voir si les machines ont accès à Internet : `ping 8.8.8.8`. La machine répond.

Pour effectuer des redirections de ports, on va dans les configurations de la machine, dans l'onglet "Réseau", sur la carte réseau 1, "avancé" et on clique sur "Redirection de ports". On ajoute un règle, on renseigne le port hote : 2222 pour le NodeManager, 2223 pour le serveur Web et 2224 pour le serveur de BDD. On renseigne le port 22 pour le port invité (le port SSH). 

## Exercice 2 : Installation et configuration d’Ansible

### Question 4 : Installation

Pour installer Ansible à partir d'un PPA, on exécute les commandes :
`$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible`

### Question 5 : Utilisateur user-ansible sur le node manager

Pour créer un utilisateur user-ansible sur le NodeManager, on exécute la commande `sudo useradd -m user-ansible --shell /bin/bash`. Puis on l'ajoute dans le groupe sudo avec la commande `sudo usermod -a -G sudo user-ansible`.
On active ce nouvel utilisateur en lui attribuant un mot de passe avec la commande : `sudo passwd user-ansible`. On renseigne le nouveau mot de passe tel que demandé par la machine : 'ansible'.
Enfin, on se connecte à ce compte utilisateur pour la suite du tp avec la commande `su user-ansible` et en renseignant le mot de passe de user-ansible.

### Question 6 : Vérification de la connexion SSH

Depuis le node manager, pour se connecter en SSH au compte admin de chaque node, on se met sur le NodeManager et on exécute les commandes `ssh tp@http1` et `ssh tp@bdd1`, ce qui suit ssh correspond au nom des machines sur lesquelles on veut se connecter. On tape les mots de passe lorsque demandé. La connexion SSH se fait.

### Question 7 : Inventaire des nodes




Petit hors-sujet : pour activer la coloration syntaxique, on exécute les commandes `nano .bashrc`, on décommente la ligne `force_color_prompt=yes`. Puis on exécute la configuration avec la commande `source .bashrc`.
