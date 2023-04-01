# ansible_lab

.- En el fichero inventory se especifican los servidores que queremos manejar
 
check all servers in inventroy file

````
ansible all -i inventory -m ping
````

mismo comando especificando la key a utilizar

````
ansible all --key-file ~/.ssh/ansible_lab -i inventory -m ping
````

comando más simple una vez añadido el fichero ansible.cfg que contiene referencia al fichero de inventory y la key a utilizar

````
ansible all -m ping
````

comando para listar todos los targets que tenemos definidos

````
ansible all --list-host
````

comando para obtener toda la información de los targets

````
ansible all -m gather_facts
````

mismo comando para obtener información de un target en concreto

````
ansible all -m gather_facts --limit ansible@192.168.1.180
````

comando para ejecutar una operación en los targets que necesita de sudo y su contraseña

````
ansible all -m apt -a update_cache=true --become --ask-become-pass
````

comando para instalar un paquete en los servidores (aunque en local te exige presionar Y, con ansible no lo hace)

````
ansible all -m apt -a name=screenfetch --become --ask-become-pass
````

Install a package via the apt module, and also make sure it’s the latest version available (similar a apt update)

````
ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass
````

Upgrade all the package updates that are available

````
ansible all -m apt -a upgrade=dist --become --ask-become-pass
````

ejecutando el playbook install_apache.yaml
````
ansible-playbook --ask-become-pass install_apache.yml
````

ejecutando la v2 del playbook install_apavhe_v2.yaml que añade un paso extra antes de instalar apache que es el de actualizar los repositorios

````
ansible-playbook --ask-become-pass install_apache_v2.yml
````

ejecutando la versión 3 de install_apache_v3.yaml que añade soporte php a apache

````
ansible-playbook --ask-become-pass install_apache_v3.yml
````

ejecutando la versión 4 de install_apache_v4.yaml que fuerza a que la versón a instalar sea la más reciente, en este caso si el paquete está instalado pero hay una versión más actual, lo actualiza.

````
ansible-playbook --ask-become-pass install_apache_v4.yml
````

playbook que elimnia apache de los targets

````
ansible-playbook --ask-become-pass remove_apache.yaml
````

playbook install_apache_v5.yaml contiene una condición para que sólo haga el apt install apache2 si el target está corriendo Ubuntu

````
sts: all
   become: true
   tasks:
 
   - name: update repository index
     apt:
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
   - name: install apache2 package
     apt:
       name: apache2
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: add php support for apache
     apt:
       name: libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
````

playbook install_apache_v6.yaml que actualiza no sólo Ubuntu, sino cualquier distribución que esté basada en Debian, (que su gestor de paquetes sea apt)

````
sts: all
  become: true
  tasks:

  - name : update repository index
    apt:
      update_cache: yes
    when: ansible_distribution in ["Debian", "Ubuntu"]

  - name: install apache2 package
    apt:
      name: apache2
    when: ansible_distribution in ["Debian", "Ubuntu"]

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
    when: ansible_distribution in ["Debian", "Ubuntu"]

````

comando para obtener la distribución que está usando el target que le especifiquemos

````
ansible all -m gather_facts --limit ansible@192.168.1.180 | grep ansible_distribution
````
playbook install_apache_Ubuntu_Fedora.yaml que instala apache en los servidores Ubuntu y Fedora

````
- hosts: all
  become: true
  tasks:
 
  - name: update repository index
    apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"
 
  - name: install apache2 package
    apt:
      name: apache2
      state: latest
    when: ansible_distribution == "Ubuntu"
 
  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"
 
  - name: update repository index
    dnf:
      update_cache: yes
    when: ansible_distribution == "Fedora"
 
  - name: install httpd package
    dnf:
      name: httpd
      state: latest
    when: ansible_distribution == "Fedora"
 
   - name: add php support for apache
    dnf:
      name: php
      state: latest
    when: ansible_distribution == "Fedora"
````

````
ansible-playbook --ask-become-pass install_apache_Ubuntu_Fedora.yml
````

modificado playbook remove_apache.yaml para que sólo afecte a los targets Ubuntu

````
- hosts: all
  become: true
  tasks:

  - name: remove apache2 package
    apt:
      name: apache2
      state: absent
    when: ansible_distribution == "Ubuntu"

  - name: remove php support for apache
    apt:
      name: libapache2-mod-php
      state: absent
    when: ansible_distribution == "Ubuntu"
````

añadido playbook remove_apache_Fedora.yaml para que sólo afecte a los target Fedora

````
- hosts: all
  become: true
  tasks:

  - name: remove apache2 package
    dnf:
      name: httpd
      state: absent
    when: ansible_distribution == "Fedora"

  - name: remove php support for apache
    dnf:
      name: php
      state: absent
    when: ansible_distribution == "Fedora"
````

añadido playbook install_apache_Ubuntu_Fedora_condensed.yaml que utiliza listas para instalar los paquetes en un sólo step en lugar de un stop por cada paquete que deseamos instalar en los targets

````
 - hosts: all
   become: true
   tasks:
 
   - name: update repository index
     apt:
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
   - name: install apache2 and php packages for Ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: update repository index
     dnf:
       update_cache: yes
     when: ansible_distribution == "Fedora"
 
   - name: install apache and php packages for Fedora
     dnf:
       name:
         - httpd
         - php
       state: latest
     when: ansible_distribution == "Fedora"
````

añadido playbook install_apache_Ubuntu_Fedora_condensed_plus.yaml que condensa aún más el playbook para instalar apache y php en los targets pq se ha eliminado el step del apt update/dnf update añadiendo la instrucción update_cache: yes en los steps

````
- hosts: all
   become: true
   tasks:
 
   - name: install apache2 package
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
   - name: install httpd package
     dnf:
       name:
         - httpd
         - php
       state: latest
       update_cache: yes
     when: ansible_distribution == "Fedora"
````

añadido playbook que hace uso de variables para definir los paquetes a instalar en los targets

````
 - hosts: all
   become: true
   tasks:
 
   - name: install apache and php
     package:
       name:
         - {{ apache_package }}
         - {{ php_package }}
       state: latest
       update_cache: yes

````
 
este playbook va acompañado de este invetory_with_package que es donde se definen los valores de las variables

````
ansible@192.168.1.180 apache_package=apache2 php_package=libapache2-mod-php
ansible@192.168.1.86 apache_package=apache2 php_package=libapache2-mod-php
ansible@192.168.1.184 apache_package=apache2 php_package=libapache2-mod-php
ansible@192.168.1.189 apache_package=httpd php_package=php
````

````
ansible-playbook -i inventory_with_package --ask-become-pass install_apache_Ubuntu_Fedora_condensed_plus+.yaml
````

playbook install_apache_v7.yaml no condesando en un solo step, vuelve a tener dos steps, uno para los targets Ubuntu y otro para los Fedora

````
 ---
 
 - hosts: all
   become: true
   tasks:
 
   - name: install apache and php for Ubuntu servers
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
   - name: install apache and php for Fedora servers
     dnf:
       name:
         - httpd
         - php
       state: latest
       update_cache: yes
     when: ansible_distribution == "Fedora"
````

añadido nuevo fichero inventory_withgroup, el objetivo es no instalar siempre lo mismo en todos los targets, estos grupos se llaman nodos y dependiendo del grupo al que pertenezca puedes especificar qué deseas instalar en ellos.s

````
[web_servers]
ansible@192.168.1.180
ansible@192.168.1.86

[db_servers]
ansible@192.168.1.184

[file_servers]
ansible@192.168.1.189
````

añadido playbook site.yaml que instala apahe y php en el grupo web_servers definido en inventory_with_groups

````
 - hosts: all
   become: true
   tasks:
 
   - name: install updates (Fedora)
     dnf:
       update_only: yes
       update_cache: yes
     when: ansible_distribution == "Fedora"
 
   - name: install updates (Ubuntu)
     apt:
       upgrade: dist
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
 
 - hosts: web_servers
   become: true
   tasks:
 
   - name: install apache and php for Ubuntu servers
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: install apache and php for Fedora servers
     dnf:
       name:
         - httpd
         - php
       state: latest
     when: ansible_distribution == "Fedora"

````

hay que indicar el fichero nuevo inventory_with_groups pq por defecto utiliza el que está definido en ansible.cfg, el cual no tiene grupos definidos

````
ansible-playbook --ask-become-pass site.yaml -i inventory_with_groups
````

se añade el playbook site2.yaml que es básicamente igual al site.yaml salvo que por buenas prácticas las tareas que quieras que siempre se ejecuten antes que el resto deben estar bajo la etiqueta pre-tasks en lugar de task, en este caso, nos interesa que siempre realice el apt update o el dnf update como primer paso antes de instalar cualquier paquete q le indiquemos.

````
---
 
 - hosts: all
   become: true
   pre_tasks:
 
   - name: install updates (Fedora)
     dnf:
       update_only: yes
       update_cache: yes
     when: ansible_distribution == "Fedora"
 
   - name: install updates (Ubuntu)
     apt:
       upgrade: dist
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
 
 - hosts: web_servers
   become: true
   tasks:
 
   - name: install apache2 package
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: install httpd package
     dnf:
       name:
         - httpd
         - php
       state: latest
     when: ansible_distribution == "Fedora"
````

añadimos playbook site3.yaml donde se crean tareas para el gupo db_servers

````
 ---
 
 - hosts: all
   become: true
   pre_tasks:
 
   - name: install updates (Fedora)
     dnf:
       update_only: yes
       update_cache: yes
     when: ansible_distribution == "Fedora"
 
   - name: install updates (Ubuntu)
     apt:
       upgrade: dist
       update_cache: yes
    when: ansible_distribution == "Ubuntu"
 
 
 - hosts: web_servers
   become: true
   tasks:
 
   - name: install httpd package (Fedora)
     dnf:
       name:
         - httpd
         - php
       state: latest
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
 - hosts: db_servers
   become: true
   tasks:
 
   - name: install httpd package (Fedora)
     dnf:
       name: mariadb
       state: latest
     when: ansible_distribution == "Fedora"
 
   - name: install mariadb server
     apt:
       name: mariadb-server
       state: latest
     when: ansible_distribution == "Ubuntu"

````

````
ansible-playbook --ask-become-pass site3.yaml -i inventory_with_groups
````

una forma de averiguar si mariadb se instaló en los servers del grupo db_servers es conectándote al servidor por ssh y ejecutando este comando:

````
systemctl status mariadb
````

añadido playbook site4.yaml que añade una task para instalar un servidor de archivos samba en el grupo de servidores file_servers

````
 - hosts: all
   become: true
   pre_tasks:
 
   - name: install updates (Fedora)
     dnf:
       update_only: yes
       update_cache: yes
     when: ansible_distribution == "Fedora"
 
   - name: install updates (Ubuntu)
     apt:
       upgrade: dist
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
 
 - hosts: web_servers
   become: true
   tasks:
 
   - name: install httpd package (Fedora)
     dnf:
       name:
         - httpd
         - php
       state: latest
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
 - hosts: db_servers
   become: true
   tasks:
 
   - name: install mariadb server package (Fedora)
     dnf:
       name: mariadb
       state: latest
     when: ansible_distribution == "Fedora"
 
   - name: install mariadb server
     apt:
       name: mariadb-server
       state: latest
     when: ansible_distribution == "Ubuntu"
 
 - hosts: file_servers
   become: true
   tasks:
 
   - name: install samba package
     package:
       name: samba
       state: latest

````

playbook site_with_tags.yaml que incluye tags para sólo contactar con los targets que estén bajo ese tag, hasta ahora los playbooks que teníamos contactaban con todos los targets o con todos los que pertenecian a un grupo y sólo actuaban si la distro era la que indicabamos en el condicional.

````
 - hosts: all
   become: true
   pre_tasks:
 
   - name: install updates (Fedora)
     tags: always
     dnf:
       update_only: yes
       update_cache: yes
     when: ansible_distribution == "Fedora"
 
   - name: install updates (Ubuntu)
     tags: always
     apt:
       upgrade: dist
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
 
 - hosts: web_servers
   become: true
   tasks:
 
   - name: install httpd package (Fedora)
     tags: apache,fedora,httpd
     dnf:
       name:
         - httpd
         - php
       state: latest
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     tags: apache,apache2,ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
 - hosts: db_servers
   become: true
   tasks:
 
   - name: install mariadb server package (Fedora)
     tags: fedora,db,mariadb
     dnf:
       name: mariadb
       state: latest
     when: ansible_distribution == "Fedora"
 
   - name: install mariadb server
     tags: db,mariadb,ubuntu
     apt:
       name: mariadb-server
       state: latest
     when: ansible_distribution == "Ubuntu"
 
 - hosts: file_servers
   tags: samba
   become: true
   tasks:
 
   - name: install samba package
     tags: samba
     package:
       name: samba
       state: latest
````

comando para listar los tags disponibles en un playbook

````
ansible-playbook --list-tags site_with_tags.yaml
````

ejemplos del uso de los tags para sólo contactar con los targets que estén bajo dicho tag

````
ansible-playbook --tags db --ask-become-pass site_with_tags.yaml -i inventory_with_groups
ansible-playbook --tags fedora --ask-become-pass site_with_tags.yaml -i inventory_with_groups
ansible-playbook --tags apache --ask-become-pass site_with_tags.yaml -i inventory_with_groups
ansible-playbook --tags `"apache,fedora" --ask-become-pass site_with_tags.yaml -i inventory_with_groups
````

añadido playbooks para eliminar mariadb y samba de los targets
* remove_mariadb.yaml 
````
- hosts: all
  become: true
  tasks:

  - name: remove maiadb package Ubuntu
    apt:
      name: mariadb
      state: absent
    when: ansible_distribution == "Ubuntu"

  - name: remove mariadb package Fedora
    dnf:
      name: mariadb-server
      state: absent
    when: ansible_distribution == "Fedora"
````

````
ansible-playbook --ask-become-pass remove_mariadb.yaml 
````

* remove_samba.yaml
````
- hosts: all
  become: true
  tasks:

  - name: remove maiadb package Ubuntu
    apt:
      name: samba
      state: absent
    when: ansible_distribution == "Ubuntu"

  - name: remove mariadb package Fedora
    dnf:
      name: samba
      state: absent
    when: ansible_distribution == "Fedora"
````

````
ansible-playbook --ask-become-pass remove_samba.yaml
````

