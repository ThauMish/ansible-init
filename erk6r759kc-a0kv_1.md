
# Ansible : 

git clone [https://github.com/ThauMish/ansible-formation](https://github.com/ThauMish/ansible-formation)

    

Ansible c'est un moteur d'automatisation open source, cela élimine toutes les corvées répétitives.

    

Cela permet aussi d’améliorer l'évolutivité , la cohérence et la fiabilité de notre environnement informatique.

    

Trois types de tâches : 

        

Provisioning: configurer les différents serveurs dont on a besoin dans notre infrastructure.

    

Configuration: modifier la configuration d'une applications , d'un os (système d'exploitation) , ou même d'un périphérique. On peux démarrer et arrêter les services, installer ou mettre à jours des applications...

    

Déploiement d'applications: toujours en adoptant une démarche devops qui va permettre d'automatiser ce déploiement d’applications.

    

### Comment fonctionne Ansible : 

    

Il existe deux catégories de "nœud", on va avoir le nœud maitre et les nœuds esclaves.

Le nœud maître est une machine sur laquelle est installé l'outil ansible. Il doit y avoir au moins un nœud maître.



Ansible fonctionne en se connectant à nos nœuds en SSH , et en y poussant de petits programmes, appelés modules. Ces modules sont définis dans un fichier nommé le Playbook. 



Le nœud maître, se base sur un fichier d'inventaire qui fournit la listes des hôtes sur lesquels les modules ansible doivent être exécutés.



Ansible exécute ces modules en SSH et les supprime une fois terminé, La seule condition requise pour cette interaction est que votre nœud maitre ansible dispose d'un accès de connexion aux nœuds esclaves.



[https://hedgedoc.dawan.fr/uploads/upload\_00dd52c1e6e7b9bead2d1822b6601b7f.png](https://hedgedoc.dawan.fr/uploads/upload\_00dd52c1e6e7b9bead2d1822b6601b7f.png)

## 

### Environnement 



Nœud-m

4 GB de ram

1 vcpu

25GB de disque

Ubuntu v20.04-6-server



Nœud-1

2 GB de ram

1 vcpu

25GB de disque

Ubuntu v20.04-6-server



Nœud-2

2 GB de ram

1 vcpu

25GB de disque

Ubuntu v20.04-6-server



ssh-keygen 

ssh-copy-id user@host



De notre serveur maitre on copie notre clé sur les noeuds esclaves.



### Composant clés : 

    

Node-manager (noeud maitre) : Aussi "control-node" c'est le post sur lequel tout est executié via des connexions en ssh aux noeuds cibles.



Playbook : Un playbook ansible décrit une suite de taches ou de rôles ecrits dans un fichier au format yaml.



Rôle : Afin d’éviter d'ecrire encore et encore le même code dans les playbooks, Ansible permet d'utiliser des librairies regroupant des fonctionnalités spécifiques. Ces librairies sont appelées des rôles qui peuvent être utilisés dans les playbooks.

    

Inventory : La description des systèmes ciblé et géré par ansible est appelé un inventaire. A chaque nœud de mon inventaire je peux y attribuer des variables.



Inventaire statique constitué de fichier plat.

Inventaire dynamique fournis par un système centralisé, AWS EC2.

    

Module : Les taches et les rôles font appel à des modules installés avec ansible.

[https://docs.ansible.com/ansible/2.9/modules/list\_of\_all\_modules.html](https://docs.ansible.com/ansible/2.9/modules/list\_of\_all\_modules.html).

    

Template : Un modèle permettant de générer un fichier cible. Ansible utilise Jinja2 qui écrit en python qui permet de gérer des modèles. Des boucles , des tests logiques des listes ou variables. 

    

Notifier : Indique que si une tache change d'etat , (et uniquement si la tache a engendré un changement, notify fait appel au handler associé pour exécuter une autre taches.

    

Handler : Tache qui n'est appelée que dans l’éventualité ou une notif est invoquée.



### Installation d'ansible (nos 3 noeuds): 

    

Il existe différentes solutions pour installer Ansible :

    

- Installation de la dernière version proposée par notre gestionnaire de package de notre os (apt install ansible).



- Installation avec le gestionnaire de paquets python pip.



- Installation depuis les sources ansible directement pour tester les nouvelles fonctionnalités.



### Installer avec pip : 

- Verif installation python : python3 -v

- Verif installation pip : python3 -m pip -V



Vérifier que toutes les commandes soient exécutées par le même utilisateur sur chaque nœuds.

curl [https://bootstrap.pypa.io/get-pip.py](https://bootstrap.pypa.io/get-pip.py) -o get-pip.py

python3 get-pip.py --user



Install ansible : 

    

python3 -m pip install --user ansible

export PATH=$PATH:/home/user1/.local/bin



ansible --version



Notre utilisateur possède un mot de passe gênant pour l’exécution de commandes en tant que sudoers, on modifie donc ce fichier pour accepter le fait que notre user n'ai pas de mot de passe.



    echo "$USER ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee "/etc/sudoers.d/dont-prompt-$USER-for-sudo-password"



## Lancement de notre première commande playbook (noeud maitre) : 

    

### L'inventaire : 

    

Ansible lit les informations sur les machines que l'on souhaite gérer à partir d'un fichier nommé l'inventaire.



Dans notre inventaire on peut stocker les adresse ips, ou les noms de domaines complets. 



On se créer un dossier "ansible-formation" avec a l’intérieur un dossier "inventaires" comportant notre fichier "host"

    

vi inventaire/hosts :

    

[ubuntu]

noeud-1 ansible\_host=192.168.x.x

noeud-2 ansible\_host=192.168.x.x



On indique à ansible l'emplacement de notre inventaire, en le précisant dans ansible.cfg.



(Option 1)

Pour avoir toutes les options d'ansible générées de manière désactiver :

     

ansible-config init --disabled > ansible.cfg



(Option 2 + simple)

vi ansible.cfg



[defaults]

inventory = inventaires/host



### Nos premières commandes ansible : 

    

Quesqu'un module ? Les modules sont comme de petits programmes qu'ansible pousse depuis une machine de controle (noeud maitre) vers tous les hotes distants (noeud esclaves). Les modules sont executé a l'aide de playbook ou directement en ligne de commande.



Module ping : envoie une requete ping a tous les noeuds de notre inventaires.



ansible all -m ping -u *user1*



- all : ici on demande a ansible d'executer la commandes sur tous les hotes de notre inventaires.

-m ping : ici on lui demander d'utiliser le module ping

-u user1: ici on lui demande de lancer notre module depuis l'utilisateur user1



noeud-1 | SUCCESS => {

    "ansible\_facts": {

        "discovered\_interpreter\_python": "/usr/bin/python3"

    },

    "changed": false,

    "ping": "pong"

}

noeud-2 | SUCCESS => {

    "ansible\_facts": {

        "discovered\_interpreter\_python": "/usr/bin/python3"

    },

    "changed": false,

    "ping": "pong"

}



En vert : la situation a etais respecter, aucun changement d'appliqué sur nos différents noeuds

En jaune : Un changement a etais effectuer sur nos noeuds

En rouge : Une erreur



Attention au module raw ! 

ansible all -m raw -a "whereis python" -u user1



Qui ne permet pas de travailler sur l'etat de nos noeuds.



Certaine commande necessite des droits de "sudoers" une elevation de privilège : 

    

ansible all -a "grep user1 /etc/shadow" -u user1 



noeud-1 | FAILED | rc=2 >>

grep: /etc/shadow: Permission deniednon-zero return code

noeud-2 | FAILED | rc=2 >>

grep: /etc/shadow: Permission deniednon-zero return code



Pour devenir sudoers , on doit activer l'option "become".



ansible all -a "grep user1 /etc/shadow" -u user1 -b



noeud-2 | CHANGED | rc=0 >>

user1:$6$LBBqI5zRgXT/Z2VU$wir4YSe3LWdgbD3eWaNFkdwcy.PkZUP8hYI.CQyaABrLnzahDEuuVOHUoen2yIQZLUM4honoh.fwAgYiJwiLA.:19471:0:99999:7:::

noeud-1 | CHANGED | rc=0 >>

user1:$6$uLm3x95m0JfB2r8a$3rPFr8oXIFPKIl/E5GcKuJWzGlOLw.xIvybZTT.3R9gHgwo61K5rE9pdK5cL0LCs6wVm2ZHOSBxU/BxujvhXa1:19471:0:99999:7:::

    

Utilisateur par défaut, peut-etre ajouter au fichier host pour faciliter les prochaines commandes sans preciser a chaque fois l'user.



[ubuntu]

noeud-1 ansible\_host=192.168.31.135 ansible\_user=user1

noeud-2 ansible\_host=192.168.31.22 ansible\_user=user1



On va devenir sudoers de base.

nano ansible.cfg



[defaults]

inventory = inventaires/host



[privilege\_escalation]

become = true

become\_user = root



ansible all -a "grep user1 /etc/shadow"





### Les groupes dans l'inventaires : 

    

On peut assigner un groupe ou plusieurs groupes pour chaque noeuds.

    

[ubuntu]

noeud-1 ansible\_host=192.168.31.135 ansible\_user=user1

noeud-2 ansible\_host=192.168.31.22 ansible\_user=user1



[prod]

noeud-1 



[dev]

noeud-2



Des commandes ad-hoc



ansible ubuntu -a "grep user1 /etc/shadow"

noeud-2 | CHANGED | rc=0 >>

user1:$6$LBBqI5zRgXT/Z2VU$wir4YSe3LWdgbD3eWaNFkdwcy.PkZUP8hYI.CQyaABrLnzahDEuuVOHUoen2yIQZLUM4honoh.fwAgYiJwiLA.:19471:0:99999:7:::

noeud-1 | CHANGED | rc=0 >>

user1:$6$uLm3x95m0JfB2r8a$3rPFr8oXIFPKIl/E5GcKuJWzGlOLw.xIvybZTT.3R9gHgwo61K5rE9pdK5cL0LCs6wVm2ZHOSBxU/BxujvhXa1:19471:0:99999:7:::

    

ansible dev -a "grep user1 /etc/shadow"

noeud-2 | CHANGED | rc=0 >>

user1:$6$LBBqI5zRgXT/Z2VU$wir4YSe3LWdgbD3eWaNFkdwcy.PkZUP8hYI.CQyaABrLnzahDEuuVOHUoen2yIQZLUM4honoh.fwAgYiJwiLA.:19471:0:99999:7:::

    

ansible prod -a "grep user1 /etc/shadow"

noeud-1 | CHANGED | rc=0 >>

user1:$6$uLm3x95m0JfB2r8a$3rPFr8oXIFPKIl/E5GcKuJWzGlOLw.xIvybZTT.3R9gHgwo61K5rE9pdK5cL0LCs6wVm2ZHOSBxU/BxujvhXa1:19471:0:99999:7:::





ansible -m ping dev

ansible -m ping dev,prod

ansible -m ping ubuntu

ansible -m ping 'noeud-[1..2]'

ansible -m ping ubuntu,'!prod'



On peux modifier certains paramètre de nos modules : (voir doc)

ansible-doc *module*



Ici on viens ecraser la valeurs par défaut de data defini dans le module.

ansible -m ping -a "data=plouf" prod 



[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file\_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file\_module.html)



Un fichier appelé fichier1 crée dans /home/user1 avec les droits 0644



ansible all -m ansible.builtin.file -a ".........."

ansible ubuntu -m file -a "path=/home/user1/fichier1 state=touch mode=0644"



Ubuntu : correspond au groupe d'inventaire

-m ping: on fais apelle au module ping

-a "path=/home/user1/fichier1 state=touch mode=0644" : Permet de modifier/ajouter les paramètres de mon module.





### Parallelisme :

Par défaut ansible utilise seulement 5 processus simultané. Cependant si on a plus d'hote que la valeur definie par défaut.



Ansible prendra plus de temps pour traiter la requêtes sur tous les hotes.



ansible ubuntu -m ping -f 8



### Recueillir des informations (facts) sur vos hotes :

    

Les facts sont des propriétés système qui sont collecté par ansible lors de son execution sur un sytème distant (nos noeuds esclaves). Les facts contiennent des details tels que le stockage, l'os, configuration du réseau.



ansible all -m setup

ansible all -m setup -a 'filter=*ip*'



noeud-2 | SUCCESS => {

    "ansible\_facts": {

        "ansible\_all\_ipv4\_addresses": [

            "192.168.31.22"

        ],

        "ansible\_all\_ipv6\_addresses": [

            "fe80::a00:27ff:fe8c:e4e"

        ],

        "ansible\_default\_ipv4": {

            "address": "192.168.31.22",

            "alias": "enp0s3",

            "broadcast": "192.168.31.255",

            "gateway": "192.168.31.1",

            "interface": "enp0s3",

            "macaddress": "08:00:27:8c:0e:4e",

            "mtu": 1500,

            "netmask": "255.255.255.0",

            "network": "192.168.31.0",

            "prefix": "24",

            "type": "ether"

        },

        "ansible\_default\_ipv6": {},

        "ansible\_fips": false,

        "discovered\_interpreter\_python": "/usr/bin/python3"

    },

    "changed": false

}



### Notion d'état et d'idempotence : 



L'idée la plus important avec ansible est de decrire un état, une configuration attendu, souhaitée. A charge pour ansible. Si la demande est respecter alors il n'y a rien a faire, la ou le module raw,shell et commandes afficherais une "erreur".



Micro-tp1 : 

    

Creation du groupe commun



ansible ubuntu -m group -a "name=commun state=present"



Le state dans ce module peut-être :

    

- present

- absent



Creation du repertoire /usr/local/partage



ansible ubuntu -m file -a "path=/usr/local/partage group=commun mode=0777 state=directory"



Le state dans ce module peut-être :

    

   * "absent"
   * "directory"
   * "file"
   * "hard"
   * "link"
   * "touch"


Affectation du groupe commun au repertoire /usr/local/partage



ansible ubuntu -m raw -a "chgrp commun /usr/local/partage"



Verif par ls -la du repertoire /usr/local/partage



ansible ubuntu -m raw -a "ls -la /user/local/partage"



### Les variables : 

    

➜  ansible-formation git:(main) ✗ cat host\_vars/noeud-1.yml 

ansible\_host: 192.168.31.135 



➜  ansible-formation git:(main) ✗ cat host\_vars/noeud-2.yml

ansible\_host: 192.168.31.22 



➜  ansible-formation git:(main) ✗cat group\_vars/ubuntu.yml

ansible\_user: user1



➜  ansible-formation git:(main) ✗ cat inventaires/host     

[ubuntu]

noeud-1

noeud-2



[prod]

noeud-1 



[dev]

noeud-2



Il y a plusieurs types de variables : 

    

- Les variables d'actes : ce sont des variables definie au sein d'un acte, invisible depuis tout autre acte.

- Les variables d'inventaires : sont definies au niveau des hosts ou des groupes dans les fichiers d'inventaires host\_vars/ et groups\_vars/.



Priorité croissant des variables : 

    

- variable de groupe d'inventaire.

- variable de groupe dans les fichiers du repertoire group\_vars.

- variable d'host dans l'inventaire.

- variable d'host dans les fichiers du repertoire host\_vars.

- variables d'actes.

- extra variable.





Variables magique : 

    

Ce sont des variables créées et gérées automatiquement par ansible :

inventory\_hostname

playbook\_dir

inventory\_dir





### Les playbooks : 

[https://www.yamllint.com/](https://www.yamllint.com/)

Les playbooks sont exprimé en YAML, un minimum de syntaxe a respecter, de ne pas etre un language de programmation ou un script, un modèle de configuration. On peux y integrer des boucles et des conditions.



Les playbooks contiennent des taches qui sont executées sequentiellement par l'utilisateur sur une machine particulière (ou plusieurs), taches dans le playbook = appelle d'un module ansible 



playgroups.yml

    ---

    - name: Creer le groupe stagiaire et le repertoire stage qui lui est associer

      hosts: ubuntu

    

      tasks:

    

        - name: Creer le groupe stagiaire

          group:

            name: stagiaire

            state: present

    

        - name: 

          file:

            path: /usr/local/stagiaire

            state: directory

            group: stagiaire

            mode: '0644'  

        

Pour executer le playbook :

    

ansible-playbook *..*.yml

ansible-playbook playgroups.yml



TP - 2 : 

    

- Adapter l'inventaire pour que le noeud-1 appartiennent au group web.



- Creer un playbook pour installer le paquet apache2 sur le groupe web.

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt\_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt\_module.html)

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/package\_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/package\_module.html)

- Ajouter une tache au playbook pour s'assurer que apache2 est bien demarré + demarrage automatique.

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd\_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd\_module.html)



    ---

    - name: Install apache

      hosts: web

    

      tasks:

    

        - name: Installation paquet apache2

          package:

            name: apache2

            state: present

    

        - name: s'assurer que le service apache2 est actif

          systemd:

            name: apache2

            enabled: yes

            state: started



Micro - tp3: 

    

A l'aide de ce module [https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile\_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile\_module.html)



Ajouter une ligne 'Listen 8080' dans le fichier qui se situe dans /etc/apache2/ports.conf



    ---

    - name: Modif port

      hosts: web

    

      tasks:

    

        - name: Add Listen 8080 to ports.conf

          lineinfile:

            path: /etc/apache2/ports.conf

            line: 'Listen 8080'



### Les handlers : 

    

Les handlers sont des listes de taches, qui ne diffère pas vraiment des taches normales, mais ils sont reférencé de manière unique et qui sont declenché par le mot clés notify.



---

    - name: Modif port

      hosts: web

    

      tasks:

    

        - name: Add Listen 8080 to ports.conf

          lineinfile:

            path: /etc/apache2/ports.conf

            line: 'Listen 8080'

          notify: restart-apache



      handlers:

        - name: restart-apache

          service:

            name: apache2

            state: restarted





Exemple d'inclusion de variable : 

    

    ---

    - name: Demo copy variable

      hosts: web

      vars\_files:

        - vars/default.yml

    

      tasks:

    

        - name: copie la variable hhtp\_port dans le fichier /tmp/port.txt

          copy:

            content: "{{ http\_port }}"

            dest: /tmp/port.txt





Exemple module debug : 

    

    ---

    - name: Exemple debug

      hosts: web

      vars\_files:

        - vars/default.yml

    

      tasks:

    

        - name: exemple module debug

          debug:

            msg: "{{ http\_port }}"





### Ansible Vault : 

    

Ansible vault est une fonctionnalité d'ansible qui nous permet de conserver des données sensibles telles que des mots de passe, des clés ssh, des certificats.



Pour creer un fichier chiffré : 

    

    ansible-vault create vars/mysql\_users.yml



---

mysql\_user: "admin"

mysql\_password: "azerty"





































































































































































































Si je tente de l'afficher sans passer par le ansible-vault 



$ANSIBLE\_VAULT;1.1;AES256

32633636363336633365323334323039346366386663386165333561356339373366333335303038

3239313531393637386434316638633033346534633332330a633230636363313165343038636132

30303865626263306639383363333661666462326530303561363334623161383162386439323931

3564356430313036390a626464326263356466623530366362636638333636386136636366643264

65636335656561336564623764633936356133313933643161393963663136613735396265663134

34316132316539303864613165636266343535363939393039373830356436646435633033626139

65313838393234353434303137653439303132393237653931386636653861326164323566333538

63666233383138333863



Pour editer un fichier chiffrer : 



    ansible-vault edit vars/mysql\_users.yml



Pour modifier le mot de passe :

    

    ansible-vault rekey vars/mysql\_users.yml



    ansible-vault encrypt vars/mysql\_users.yml

    ansible-vault decrypt vars/mysql\_users.yml

    ansible-vault view vars/mysql\_users.yml



Faire apelle a un playbook en lui demandant de demander le mot de passe du vault.

    ansible-playbook playbook-copy.yml --ask-vault-pass 



Sinon ecrire nos mot de passe dans un fichier et le passer en paramètre de l'execution de la commande playbook



    echo '*mdp*' > .vault\_pass

    ansible-playbook playbook-copy.yml --vault-password-file=.vault\_pass



ou sinon dans ansible.cfg : 

    

    [defaults]

    inventory = inventaires/host

    vault\_password\_file = .vault\_pass

    [privilege\_escalation]

    become = true

    become\_user = root



### Structure conditionelle : 

    

Clause when : Executer une tache si une condition est rempli.

Block when : Executer plusieurs si une condition est rempli.

Module assert : permet de verifier les pre-requis.

Module fail : permet l'arret d'un acte en fonction d'une ou plusieurs conditions.



### Structure itérative : 

    

Loop : qui va permettre de boucler sur notre valeurs/variables.

with\_*



    ---

    - name: Exécution conditionnelle

      hosts: web

    

      vars:

        go1: true

        go2: false

    

      tasks:

        - name: A executer si la variable go est vrai

          debug:

            msg: "ok (go= {{ go1 }})"

          when: go1 == true

    

        - name: A executer si la variable go1 et go2 sont vrai

          debug:

            msg: "ok"

          when: ansible\_os\_family == "Debian"





TP CLAUSE WHEN :

    

Commande pour faire toutes les installent centos:     



En fonction de la distribution ou le playbook est jouais soient on a un "apt install apache2" ou "yum install httpd".



Echange de clé ssh !

Installation pip sur centos !

Sudoers centos ! 

Changement dans l'inventaire !







    ---

    - name: Installer Apache2

      hosts: all

      become: true

      

      tasks:

        - name: Installer Apache2 sur les systèmes Debian

          apt:

            name: apache2

            state: latest

          when: ansible\_os\_family == 'Debian'

      

        - name: Installer Apache2 sur les systèmes CentOS

          yum:

            name: httpd

            state: latest

          when: ansible\_os\_family == 'RedHat'



TP GROUP\_VARS



Dans les fichiers groups\_vars/ubuntu.yml et centos.yml definir une variable contenant le bon paquet a installer.



Et reecrire le playbook correspondant.



cat group\_vars/ubuntu.yml



package: apache2



cat group\_vars/centos.yml



package: apache2



    

    ---

    - name: install apache

      hosts: all

    

      tasks:

        - name: Installation paquet apache2

          package:

            name: "{{ package }}"

            state: present













TP vars : 



vars/defaults.yml



groupes:

  - nom: dev

  - nom: ops

    

utilisateurs:

    - nom : toto

       groupe: dev

     - nom : titi

       groupe: ops

    



Exemple



    ---

    

    - name: Creation groupe 

      hosts: noeud-1

    

      vars\_files:

        - vars/default.yml

    

      tasks:

    

        - name: Debug vars

          debug:

            msg: "{{ item.nom }}"

          loop: "{{ groupes }}"



Dans vars/defaults.yml vous allez definir la variable groupes avec un nom associé, mais aussi créer la variable utilisateur avec un nom et groupe associé a chaque utilisateur.



Deux taches , la première tache visent a creer une liste de groupe dans sur nos noeuds 

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group\_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group\_module.html)



La deuxième taches visent a creer des utilisateurs avec un groupe associé.

[https://docs.ansible.com/ansible/latest/co:wq](https://docs.ansible.com/ansible/latest/co:wq)!llections/ansible/builtin/user\_module.html



BONUS :

Rajouter un user ou groupe en tant que sudo dans le fichier /etc/sudoers

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile\_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile\_module.html)



Fichier variables :

    

    groupes:

      - nom: dev

      - nom: ops

    

    utilisateurs:

      - nom: toto

        groupe: dev

      - nom: titi

        groupe: dev

      - nom: tata

        groupe: ops

   * 

Correction playbook :

    

    - name: Création des groupes et des utilisateurs

      hosts: all

      

      vars\_files:

        - vars/default.yml

      

      tasks:

        - name: Créer les groupes

          group:

            name: "{{ item.nom }}"

          loop: "{{ groupes }}"

      

        - name: Créer les utilisateurs

          user:

            name: "{{ item.nom }}"

            group: "{{ item.groupe }}"

          loop: "{{ utilisateurs }}"

    

        - name: Ajouter les users en tant que sudo

          lineinfile:

            path: /etc/sudoers

            line: "{{ item.nom }} ALL=(ALL) NOPASSWD:ALL"

            state: present

            validate: 'visudo -cf %s'

          loop: "{{ utilisateurs }}"

 



Exemple loop\_var index\_var



    - name: Les boucles

      hosts: noeud-1

    

      vars\_files:

        - vars/default.yml

    

      tasks:

    

        - name: Boucle 1

          debug:

            msg: "Couleur numero : {{ i+1 }} : {{ item }}"

          loop:

            - rouge

            - orange

            - bleu

          loop\_control:

            index\_var: i

    

        - name: Boucle 2 

          debug:

            msg: "Groupe numero : {{ i+1 }} : {{ ngroup }}"

          loop: "{{ groupes }}"

          loop\_control:

            index\_var: i

            loop\_var: ngroup



Templating jinja2 : 

    

Le module template, va permettre de transferer des fichiers après les avoirs soumis au moteur de templating jinja2



Ce moteur il permet de traiter des instructions inserer dans le fichier a transmettre :

    

- Instruction de test 

- structure itérative

- affectations de variables

- commentaire



[https://jinja.palletsprojects.com/en/3.0.x/templates/](https://jinja.palletsprojects.com/en/3.0.x/templates/)



Fichier txt.j2





    Mes utilisateurs sont test 1:

    

    {% for utilisateur in utilisateurs %}

    

    {{ utilisateur.nom }}

    {{ utilisateur.groupe }}

    

    {% endfor %}

    

    Mes utilisateurs sont test 2:

    

    {% for item in utilisateurs %}

    

    {{ item.nom }}

    {{ item.groupe }}

    

    {% endfor %}



Playbook : 

    

    - name: Création des groupes et des utilisateurs

      hosts: all

      

      vars\_files:

        - vars/default.yml  

    

      tasks:

    

      - name: Traitement du template

        template:

          src: utilisateurs.txt.j2

          dest: /home/user1/utilisateurs.txt





TP-1 template : 

    

1 taches



A l'aide du module template, apporter les modifications au fichiers ports.conf.j2 pour activer l'ecoute sur le port 8080, penser a ajuster votre fichier vars.



/etc/apache2/port.conf



[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template\_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template\_module.html)



    # If you just change the port or add more ports here, you will likely also

    # have to change the VirtualHost statement in

    # /etc/apache2/sites-enabled/000-default.conf

    

    Listen 80

    

    <IfModule ssl\_module>

            Listen 443

    </IfModule>

    

    <IfModule mod\_gnutls.c>

            Listen 443

    </IfModule>

    

    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet



2 eme taches.



Creer un handlers qui va redemarrer le service apache.





port.conf.j2



    # If you just change the port or add more ports here, you will likely also

    # have to change the VirtualHost statement in

    # /etc/apache2/sites-enabled/000-default.conf

    

    Listen 80

    

    <IfModule ssl\_module>

            {{ apache\_https\_port }}

    </IfModule>

    

    <IfModule mod\_gnutls.c>

            {{ apache\_https\_port }}

    </IfModule>

    {{ apache\_http\_port }}

    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet



playbook : 

    

- name: Création des groupes et des utilisateurs

  hosts: noeud-1

  

  vars\_files:

    - vars/default.yml  



  tasks:



  - name: Traitement du template

    template:

      src: files/ports.conf.j2

      dest: /etc/apache2/port.conf

    notify: restart-apache





  handlers:



    - name: restart-apache

      service:

        name: apache2

        state: restarted





2 eme TP : 

    

files/index.html.j2  





    <html>

    <head>

            <title>Liste des stagiaires</title>

    </head>

    <body>

            <h1>Liste des stagiaires</h1>

            <table>

                    <thead>

                            <tr>

                                    <th>Nom</th>

                                    <th>Prénom</th>

                                    <th>Âge</th>

                                    <th>Adresse email</th>

                            </tr>

                    </thead>

                    <tbody>

                            {% for .... in ..... %}

                            <tr>

                                    <td>{{ .... }}</td>

                                    <td>{{ .... }}</td>

                                    <td>{{ .... }}</td>

                                    <td>{{ .... }}</td>

                            </tr>

                            {% endfor %}

                    </tbody>

            </table>

    </body>

    </html>





[https://hedgedoc.dawan.fr/uploads/upload\_20df4f3b2368c0e5e796e602eef3bc17.png](https://hedgedoc.dawan.fr/uploads/upload\_20df4f3b2368c0e5e796e602eef3bc17.png)





Adapter le fichier vars/default.yml en inventant un jeu de donnée pour correspondre a "l'image"





Avec le module template appliquer les changement a ce fichier et le copier dans le noeud-1 a la place de index.html situé dans /var/www/html/



Correction : 

    

files/index.html.j2 :

    

    <!DOCTYPE html>

    <html>

    <head>

            <title>Liste des stagiaires</title>

    </head>

    <body>

            <h1>Liste des stagiaires</h1>

            <table>

                    <thead>

                            <tr>

                                    <th>Nom</th>

                                    <th>Prénom</th>

                                    <th>Âge</th>

                                    <th>Adresse email</th>

                            </tr>

                    </thead>

                    <tbody>

                            {% for apprenti in apprentis %}

                            <tr>

                                    <td>{{ apprenti.nom }}</td>

                                    <td>{{ apprenti.prenom }}</td>

                                    <td>{{ apprenti.age }}</td>

                                    <td>{{ apprenti.email }}</td>

                            </tr>

                            {% endfor %}

                    </tbody>

            </table>

    </body>

    </html>





playbook : 

    

    - name: Création des groupes et des utilisateurs

      hosts: all

      

      vars\_files:

        - vars/default.yml  

    

      tasks:

    

      - name: Traitement du template

        template:

          src: files/utilisateurs.txt.j2

          dest: /home/user1/utilisateurs.txt



vars/defaults.yml



    apprentis:

      - nom: Ballet

        prenom: Thomas

        age: 23

        email: tballet@dawan.fr

      - nom: El Harchi

        prenom: Omar

        age: 72

        email: oelharchi@dawan.fr

      - nom: Mounier

        prenom: Axel

        age: 86

        email: amounier@dawan.fr

      - nom: Andrade

        prenom: Florian

        age: 20

        email: fandrade@dawan.fr





### Les tags :

    

Les tags sont utiles pour executer certaines parties specifique d'un playbook en fonctions de vos besoins.



Les tags servent si on veux tester plusieurs ou une seule partie du playbook. On peux ajouter un ou plusieurs a une tache, un role, ou un bloc de taches.



    ---

    - name: Install apache

      hosts: ubuntu

    

      tasks:

        - name: Installation paquet apache2

          package:

            name: apache2

            state: present

          tags: ['webserver', 'apache']

    

        - name: Installation paquet curl

          package:

            name: curl

            state: present

          tags: ['webserver', 'curl']



# Executer seulement les taches avec le tag webserver : 

    ansible-playbook playbook.yml --tags webserver



# Executer seulement les taches avec le tag curl ou apache :  

    ansible-playbook playbook.yml --tags apache

    ansible-playbook playbook.yml --tags curl

    ansible-playbook playbook.yml --tags "curl,apache"



ces tags a eviter.

never unused

# Executer toutes les taches sauf une ou plusieurs

    ansible-playbook tagsbook.yml --skip-tags curl





### Dry-run



Le mode dry-run, mode simulation ou mode check, c'est une fonctionnalité d'ansible qui vous permet de verifier ce qui se passerait si on executerais notre playbook sans effectuer de changement sur nos machines cibles.

        

Pour executer le mode dry-run on dois ajouter l'option --check.

    ansible-playbook playbook.yml --check 



Pour voir la difference entre l'etat actuel et le changement d'etat une fois l'application du playbook joué : 

    ansible-playbook playbook.yml --check --diff



### Notions de Delegations



La delegation dans Ansible est un mecanisme qui permet d'exectuer une tache sur un hote différent de lui defini par la directive hosts dans le playbook. Cet fonctionnalité est souvent utilisée pour recolter des informations sur un ensemble d'hotes, et effectuer des actions sur un hote de controle ou centraliser certaines operations.



    - name: Création des groupes et des utilisateurs

      hosts: ubuntu

    

      tasks:

    

        - name: Creer un fichier de zone DNS

          template:

            src: zone.j2

            dest: /etc/bind/zones/db.ansible.com

          delegate\_to: "noeud-1" or "{{ noeud-1 }}"

          run\_once: true



Dans cet exemple, la tache "Creer un fichier de zone DNS" sera executé une seule fois sur l'hote defini par la variable noeud-1 ou l'host noeud-1. L'option run\_once garantit que la tache ne sera pas repeté sur les autres hosts.



### Les lookups : 

    

Les lookups sont des methodes pour accéder a des données à l'interieur ou a l'exterieur d'ansible, telles que des fichiers, des commandes shell, des variables d'environnement, ou encore des requetes DNS.



    - name: Les Lookups

      hosts: ubuntu

    

      tasks:

    

        - name: Resolution DNS

          debug:

            msg: "Information DNS : {{ lookup('dig','google.fr') }}"

    

        - name: Variable d'environnement

          debug:

            msg: "Le path est : {{ lookup('env', 'PATH') }} "

    

        - name: Contenu fichier /etc/passwd

          debug:

            msg: "Contenu fichier : {{ lookup ('file', '/etc/passwd').split('\n') }} "



Pour la tache resolution DNS, j'utilise le lookup dig pour effectuer une requete DNS et recuperer l'adresse IP correspondante a google.fr



    

### TP LAMP Final ?



Pour l'arborescence : 

    

tree

.

├── ansible.cfg

├── files

├── groups\_vars

│   ├── database

│   └── webserver

├── host\_vars

│   ├── noeud-1.yml

│   └── noeud-2.yml

├── inventaires

├── template

│   └─

└── vars

    └── default.yml





Dans ce tp on va deployer un cluster LAMP (Linux Apache Mysql Php) sur deux noeuds. Le premier noeud accueillera Apache, PHP et les autres composants necessaires, tandis que le deuxième noeud hebergera uniquement la base de donnée MySQL.



Instructions : 

    

1. Creer un playbook nommé 'cluster\_lamp.yml' Definissez deux groupes d'hotes : 'webserver' pour le premier noeud et 'database' pour le deuxième noeud.



2. Creer un fichier vars/defaults pour definir les variables suivantes : 

    Version de php

    Nom d'utilisateur Mysql

    Mot de passe mysql

    

    Configuration du virtualhost Apache.

    

    Template virtualhost 'apache\_vhost.j2' pour remplacer le fichier /etc/apache2/sites-available/000-default.conf : 



    <VirtualHost *:80>

    ServerName {{ apache\_vhost.server\_name }}

    ServerAlias {{ apache\_vhost.server\_alias }}

    DocumentRoot {{ apache\_vhost.document\_root }}

    

    <Directory {{ apache\_vhost.document\_root }}>

        Options Indexes FollowSymLinks

        AllowOverride All

        Require all granted

    </Directory>



    ErrorLog ${APACHE\_LOG\_DIR}/error.log

    CustomLog ${APACHE\_LOG\_DIR}/access.log combined

</VirtualHost>





3. Dans cluster\_lamp.yml ajoutez des taches pour installer et configurer apache et php sur le group webserver.

En utilisant une version de php (7.4) defini dans vars/defaults.yml.



4. Utiliser le fichier template pour generer la configuration du VirtualHost apache. En utilisant les variables definis dans vars/default.yml.



5. Utilisation de handlers lorsqu'on modifie un fichier de conf.



6. Dans cluster\_lamp.yml, ajouter des taches pour installer et configurer mysql sur le groupe database.



7. Utiliser le module mysql\_user et mysql\_db pour créer l'utilisateur et la base de donnée MySQL avec les informations definis dans vars/defaults.yml.



python3-mysqldb ?



8. Pour permettre d'acceder a la base de donnée en distance on dois modifier ce fichier /etc/mysql/mysql.conf.d/mysqld.cnf sur le noeud database.



    - name: Permettre de se connecter a distance

          lineinfile:

             path: /etc/mysql/mysql.conf.d/mysqld.cnf

             regexp: '^bind-address'

             line: 'bind-address = 0.0.0.0'

             backup: yes

          notify:

             - restart-mysql



- BONUS : Templater ce fichier php pour tester la connectivité entre apache et mysql : 

    

    <?php

    // Configuration de la connexion à la base de données

    $db\_host = '{{ mysql\_host }}';

    $db\_name = '{{ mysql\_db }}';

    $db\_user = '{{ mysql\_user }}';

    $db\_password = '{{ mysql\_password }}';

    

    try {

        // Création de l'objet PDO pour la connexion à la base de données

        $conn = new PDO("mysql:host=$db\_host;dbname=$db\_name", $db\_user, $db\_password);

        

        // Configurer PDO pour générer des exceptions en cas d'erreur

        $conn->setAttribute(PDO::ATTR\_ERRMODE, PDO::ERRMODE\_EXCEPTION);

    

        echo "Connexion réussie à la base de données";

    } catch (PDOException $e) {

        echo "Erreur de connexion à la base de données: " . $e->getMessage();

    }

    

    // Fermer la connexion

    $conn = null;

    ?>





Proposition de correction : 

    

cluster\_lamp.yml : 

    

    ---

    - hosts: webserver

      become: yes

      vars\_files:

        - vars/defaults.yml

    

      tasks:

        - name: Installer les paquets necessaires

          package:

            name:

              - apache2

              - "php{{ php\_version }}"

              - "php{{ php\_version }}-mysql"

              - libapache2-mod-php

            state: present

    

        - name: Copier la configuration du VirtualHost

          template:

            src: template/apache\_vhost.j2

            dest: /etc/apache2/sites-enabled/000-default.conf

          notify: restart-apache

    

        - name: Copier le fichier PHP de connexion à la base de données

          template:

            src: template/testconnexion.php.j2

            dest: /home/user1/webpage/index.php

    

    - hosts: database

      become: yes

      vars\_files:

        - vars/defaults.yml

    

      tasks:

        - name: Installer les paquets necessaires

          package:

            name:

              - mysql-server

              - mysql-client

              - python3-mysqldb

              - libmysqlclient-dev

            state: present

    

        - name: Start et enable mysql service

          service:

            name: mysql

            state: started

            enabled: yes

    

        - name: Creer la base de données MySQL

          mysql\_db:

            name: "{{ mysql\_db }}"

            state: present

    

        - name: Creer l'utilisateur MySQL

          mysql\_user:

            name: "{{ mysql\_user }}"

            password: "{{ mysql\_password }}"

            priv: "{{ mysql\_db }}.*:ALL"

            host: '%'

            state: present

    

        - name: Permettre de se connecter a distance

          lineinfile:

             path: /etc/mysql/mysql.conf.d/mysqld.cnf

             regexp: '^bind-address'

             line: 'bind-address = 0.0.0.0'

             backup: yes

          notify:

             - restart-mysql

    

      handlers:

    

        - name: restart-apache

          systemd:

            name: apache2

            state: restarted

            daemon\_reload: yes

    

        - name: restart-mysql

          service:

            name: mysql

            state: restarted



vars/default.yml



    php\_version: 7.4

    mysql\_user: admin

   * mysql\_password: azerty
    mysql\_db: db

    mysql\_host: 192.168.31.22

    

    apache\_vhost:

      server\_name: ansible-formation.com

      server\_alias: www.ansible-formation.com

      document\_root: /home/user1/webpage





### Les roles : 

    

Les roles sont une caracteristique d'ansible qui facilitent la reutilisation, favorisent d'avantage la modularisation de votre configuration et aussi simplifient l'ecriture de vos playbooks complexes en les divisant logiquement en composant reutilisables.



Les roles fournissent un cadre de reutilisation independante de variables, taches , fichiers, modèle etc.. 

Les roles pour ansible faut le voir comme des librairies qu'on telechargerais pour de la programmation.



Les librairies en programmation contiennent leurs propres variables et fonctions qu'on peux reutiliser en fonction de notre besoin.



Chaque role est essentiellement limité a une fonctionnalité particulière (ex installation apache). c'est reutilisable independamment et il n'existe aucun moyen d'executer directement les roles.



Dans votre arborescence creer le dossier roles et une fois dedans on initialise le role avec la commande :

    

    ansible-galaxy init *nomdurole*



Transformer votre tp lamp en un role.














