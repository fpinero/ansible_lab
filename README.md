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

playbook site_copying_a_file.yaml que además de instalar un paquete (apache2) también copia un fichero (index.html) en los targets

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
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

````
ansible-playbook --ask-become-pass -i inventory_with_groups site_copying_a_file.yaml
````

se añade el playbook site_workstation.yaml que instala el paquete unzip en el grupo workstation que vamos se ha añadido a inventory_with_groups y que va a contener la IP de nuestra máquina local, y en este caso copia la última versión de terraform.zip desde internet y con el flag unarchive lo descomprime (gracias a que antes hemos intalado el paquete unziop en el paso previo)  y le indicamos que lo copie en /usr/local/bin

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
 
 - hosts: workstations
   become: true
   tasks:

   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/1.4.4/terraform_1.4.4_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

````
ansible-playbook --ask-become-pass site_workstations -i inventory_with_groups
````

pequeña modificación a site_workstations.yaml para que installe dessde internet terraform en dos grupos, en el workstations y en el file_servers (que la máquina Fedora).

````
---
 
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
 
 - hosts: ["workstations","file_servers"]
   become: true
   tasks:

   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/1.4.4/terraform_1.4.4_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

añadido playbook site_launching_services.yaml que además de instalar el servidor web en la máquina Fedora, arraca el servicio. Esto no es necesario en las máquinas Ubuntu, pero de este modo automatizamos también el inicio del servicio httpd de Fedora.

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
 
 - hosts: workstations
   become: true
   tasks:
 
   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
 
   - name: start and enable httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: started
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     tags: apache,apache2,ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

````
ansible-playbook --ask-become-pass site_launching_services.yaml -i inventory_with_groups
````

* por defecto Fedora no tiene el puerto 80 abierto al exterior, por lo que aunque en el playbook se levante el servicio httpd sigue siendo necesario logarse en la máquina fedora y ejecutar el siguiente comando para exponer el puerto 80

````
sudo firewall-cmd --add-port=80/tcp
````

* esta version del playbook site_launching_services.yaml automatiza los pasos manuales a los que obligaba la anterior version, o sea, que abre el puerto 80 y recarga la configuración del firewall de Fedora

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
 
 - hosts: workstations
   become: true
   tasks:
 
   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
 
   - name: start and enable httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: started
       enabled: yes
     when: ansible_distribution == "Fedora"

   - name: Add port 80/tcp to firewalld (open port 80 on Fedora target)
     tags: httpd
     firewalld:
       port: 80/tcp
       permanent: true
       state: enabled
     when: ansible_distribution == "Fedora"
       
   - name: Reload Fedora firewalld
     tags: httpd 
     systemd:
       name: firewalld
       state: restarted
     when: ansible_distribution == "Fedora"

   - name: install apache2 package (Ubuntu)
     tags: apache,apache2,ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

playbook que cambia una línea dentro de un fichero (lineinfile) y que reinicia un servicio, en este ejemplo se va a cambiar la línea del email del administrador del website de Apache en Fedora, cat /etc/httpd/conf/httpd.conf | grep ServerAdmin

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
 
 - hosts: workstations
   become: true
   tasks:
 
   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
 
   - name: start and enable httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: started
       enabled: yes
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     tags: apache,apache2,ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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
site.yml (added ‘lineinfile’ play)
 ---
 
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
 
 - hosts: workstations
   become: true
   tasks:
 
   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/1.4.4/terraform_1.4.4_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
 
   - name: start and enable httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: started
       enabled: yes
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     tags: apache,apache2,ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: change e-mail address for admin
     tags: apache,fedora,httpd
     lineinfile:
       path: /etc/httpd/conf/httpd.conf
       regexp: '^ServerAdmin'
       line: ServerAdmin somebody@somewhere.net
     when: ansible_distribution == "Fedora"
     register: httpd
 
   - name: restart httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: restarted
     when: httpd.changed 
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

````
ansible-playbook --ask-become-pass site_launching_services_2.yaml -i inventory_with_groups
````
playbook site_add_an_user.yaml que añade el usuario simene en todos los targets

```
---
 
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
 
 - hosts: all
   become: true
   tasks:

   - name: create simone user
     tags: always
     user:
       name: simone
       groups: root
 
 - hosts: workstations
   become: true
   tasks:
 
   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/1.4.4/terraform_1.4.4_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
 
   - name: start and enable httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: started
       enabled: yes
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     tags: apache,apache2,ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: change e-mail address for admin
     tags: apache,fedora,httpd
     lineinfile:
       path: /etc/httpd/conf/httpd.conf
       regexp: '^ServerAdmin'
       line: ServerAdmin somebody@somewhere.net
     when: ansible_distribution == "Fedora"
     register: httpd
 
   - name: restart httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: restarted
     when: httpd.changed    
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

```

````
ansible-playbook --ask-become-pass site_add_an_user.yaml -i inventory_with_groups
````
 playbook site_add_an_user_and_pubkey.yaml que además de crear el usuario simone añade la clave pública a las autorize_key para poder realizar login con dicho usuario en los targets

````
---
 
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
 
 - hosts: all
   become: true
   tasks:
 
   - name: create simone user
     user:
       name: simone
       groups: root
     
   - name: add ssh key for simone
     tags: always
     authorized_key:
       user: simone
       key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILDhUqqqHvSQ6z+v5SuqkZ/voHv9ksxoMVo9sC3MFcU6 ansible_lab"
         
   - name: add  file for simone
     tags: always
     copy:
       src: sudoer_simone
       dest: /etc/.d/simone
       owner: root
       group: root
       mode: 0440
 
 - hosts: workstations
   become: true
   tasks:
 
   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/1.4.4/terraform_1.4.4_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
   
   - name: start and enable httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: started
       enabled: yes
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     tags: apache,apache2,ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: change e-mail address for admin
     tags: apache,fedora,httpd
     lineinfile:
       path: /etc/httpd/conf/httpd.conf
       regexp: '^ServerAdmin'
       line: ServerAdmin somebody@somewhere.net
     when: ansible_distribution == "Fedora"
     register: httpd
  
   - name: restart httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: restarted
     when: httpd.changed    
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

````
ansible-playbook --ask-become-pass site_add_an_user_and_pubkey.yaml -i inventory_with_groups
````

se puede comprobar que se ha creado el ususario simone y que acepta la clave ssh haciendo loging a cualquiera de los targets, por ejemplo al Fedora

````
ssh -i ~/.ssh/ansible_lab simone@192.168.1.189
````

añadiendo a ansible.cfg el nombre del usuario a emplear en los targets y dado que ya estaba añadida la key a utilizar para realizar ssh en ellos, se puede prescindir de incluir --ask-become-pass an ejecutar ansible-playbook y no tener que introducir la contraseña del usuario cada vez que se lanza el playbook, lo cual automatiza aún más el proceso.

````
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible_lab
remote_user = simone
````

indicamos el usuario </b>simone</b> porque en el playbook lo añadimos como usuario similar a root al copiar el fichero sudoer_simone donde se le indicaba que no solicitara password e indicacomos que pertenecía al propietario root y grupo root, siempre debía validarse con la key que también añadimos.

````
simone ALL=(ALL) NOPASSWD: ALL
````

````
src: sudoer_simone
       dest: /etc/.d/simone
       owner: root
       group: root
       mode: 0440
````

````
ansible-playbook site_add_an_user_and_pubkey.yaml  -i inventory_with_groups
````
lo que acabamos de realizar de ejecutar un playbook sin que pida la contraseña de un usuario con privilegios de root ha funcionado porque anteriormente ejecutamos un playbook que creaba dicho usuario y añadía sy ssh key, pero no hubiese funcionado si este fuera el primer playbook que se ejecuta en los targets. Por ello, el primer playbook debe ser una platilla inicial que cree los pre-requisitos necesarios en los targets para que posteriormente ejecutemos los playbook sin tener que introducir la contraseña del usuario. Para ello se crea un primer playbook llamado bootstrap.yaml con lo esencial para crear el usuario y de paso actualizar el sistema.

````
---
 
 - hosts: all
   become: true
   pre_tasks:
 
   - name: install updates (CentOS)
     tags: always
     dnf:
       update_only: yes
       update_cache: yes
     when: ansible_distribution == "CentOS"
 
   - name: install updates (Ubuntu)
     tags: always
     apt:
       upgrade: dist
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
 - hosts: all
   become: true
   tasks:
 
   - name: create simone user
     user:
       name: simone
       groups: root
 
   - name: add ssh key for simone
     authorized_key:
       user: simone
       key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAe7/ofWLNBq3+fRn3UmgAizdicLs9vcS4Oj8VSOD1S/ ansible"
      
   - name: add sudoers file for simone
     copy:
       src: sudoer_simone
       dest: /etc/sudoers.d/simone
       owner: root
       group: root
       mode: 0440
````

````
ansible-playbook --ask-become-pass bootstrap.yaml -i inventory_with_groups
````

del resto de playbooks debe eliminarse la parte que crea el usuario, suble la key y añade el usuario a sudoers.d

de este modo queda el playbook site_not_asking_pwd.yaml que no añade el usuario simone al grupo de root y subiendo su key cada vez que se ejecuta

````
---
 
 - hosts: all
   become: true
   pre_tasks:
 
   - name: update repository index (Fedora)
     tags: always
     dnf:
       update_cache: yes
     changed_when: false
     when: ansible_distribution == "Fedora"
 
   - name: update repository index (Ubuntu)
     tags: always
     apt:
       update_cache: yes
     changed_when: false
     when: ansible_distribution == "Ubuntu"
 
 - hosts: all
   become: true
   tasks:
 
   - name: add ssh key for simone
     authorized_key:
       user: simone
       key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILDhUqqqHvSQ6z+v5SuqkZ/voHv9ksxoMVo9sC3MFcU6 ansible_lab"
 
 - hosts: workstations
   become: true
   tasks:
 
   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
       src: https://releases.hashicorp.com/terraform/1.4.4/terraform_1.4.4_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root
 
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
 
   - name: start and enable httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: started
       enabled: yes
     when: ansible_distribution == "Fedora"
 
   - name: install apache2 package (Ubuntu)
     tags: apache,apache2,ubuntu
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: change e-mail address for admin
     tags: apache,fedora,httpd
     lineinfile:
       path: /etc/httpd/conf/httpd.conf
       regexp: '^ServerAdmin'
       line: ServerAdmin somebody@somewhere.net
     when: ansible_distribution == "Fedora"
     register: httpd
 
   - name: restart httpd (Fedora)
     tags: apache,fedora,httpd
     service:
       name: httpd
       state: restarted
     when: httpd.changed    
 
   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
 
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

```
ansible-playbook site_not_asking_pwd.yaml  -i inventory_with_groups
```
playbook site_with_roles.yaml que ayuda a que nuestros playbooks sean menos desordenados

````
---
 
 - hosts: all
   become: true
   pre_tasks:
 
   - name: update repository index (CentOS)
     tags: always
     dnf:
       update_cache: yes
     changed_when: false
     when: ansible_distribution == "CentOS"
 
   - name: update repository index (Ubuntu)
     tags: always
     apt:
       update_cache: yes
     changed_when: false
     when: ansible_distribution == "Ubuntu"
 
 - hosts: all
   become: true
   roles:
     - base
    
 - hosts: workstations
   become: true
   roles:
     - workstations
 
 - hosts: web_servers
   become: true
   roles:
     - web_servers
 
 - hosts: db_servers
   become: true
   roles:
     - db_servers
 
 - hosts: file_servers
   become: true
   roles:
     - file_servers
````

este playbook no se puede ejecutar hasta que no creemos el directorio roles y dentro de él los subdirectorios de los roles y dentro de cada uno de ellos otro  subdirectorio llamado tasks

```
mkdir roles
```

````
cd roles
mkdir base
mkdir db_servers
mkdir file_servers
mkdir web_servers
mkdir workstations
````

````
cd <role_name>
mkdir tasks
````

creemos dentro del directorio roles/base/tasks el fichero base.yaml conteniendo los datos del usuario a utilizar en los playbooks

````
- name: add ssh key for simone
     authorized_key:
       user: simone
       key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAe7/ofWLNBq3+fRn3UmgAizdicLs9vcS4Oj8VSOD1S/ ansible"
````

creemeos el fichero main.yaml en cada uno de los otros directorios tasks de cada uno de los roles con lo que queramos que haga ese playbook

````
vim ./db_servers/tasks/main.yaml
````

````
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
````

````
vim ./file_servers/tasks/main.yaml
````

````
- name: install samba package
   tags: samba
   package:
     name: samba
     state: latest
````

````
vim ./workstations/tasks/main.yaml
````

````
- name: install unzip
  package:
    name: unzip
 
- name: install terraform
  unarchive:
   src: https://releases.hashicorp.com/terraform/1.4.4/terraform_1.4.4_linux_amd64.zip
   dest: /usr/local/bin
   remote_src: yes
   mode: 0755
   owner: root
   group: root
````

````
vim ./web_servers/tasks/main.yaml
````

````
- name: install httpd package (Fedora)
  tags: apache,fedora,httpd
  dnf:
    name:
      - httpd
      - php
    state: latest
  when: ansible_distribution == "Fedora"
 
- name: start and enable httpd (Fedora)
  tags: apache,fedora,httpd
  service:
   name: httpd
   state: started
   enabled: yes
  when: ansible_distribution == "Fedora"
 
- name: install apache2 package (Ubuntu)
  tags: apache,apache2,ubuntu
  apt:
    name:
      - apache2
      - libapache2-mod-php
    state: latest
  when: ansible_distribution == "Ubuntu"
 
- name: change e-mail address for admin
  tags: apache,fedora,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "Fedora"
  register: httpd
 
- name: restart httpd (Fedora)
  tags: apache,fedora,httpd
  service:
    name: httpd
    state: restarted
  when: httpd.changed    
 
- name: copy html file for site
  tags: apache,apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644

````

así queda la estructura sedinitiva de roles

````
roles
├── base
│   └── tasks
│       └── main.yaml
├── db_servers
│   └── tasks
│       └── main.yaml
├── file_servers
│   └── tasks
│       └── main.yaml
├── web_servers
│   ├── files
│   │   └── default_site.html
│   └── tasks
│       └── main.yaml
└── workstations
    └── tasks
        └── main.yaml
````

````
ansible-playbook site_with_roles.yaml -i inventory_with_groups
````
se crea el direcorio host_vars para almacenar el fichero de variables a utilizar en este ejemplo en los targets web_servers

````
mkdir host_vars 
````

y en dicho directorio se crea un fichero yaml con el nombre de cada uno de los targets con los que trabajamos

````
ansible@192.168.1.180.yaml
ansible@192.168.1.184.yaml
ansible@192.168.1.189.yaml
ansible@192.168.1.86.yaml
````

todos los ficheros son iguales excepto el ansible@192.168.1.189.yaml que tiene las variables ajustadas a Fedora

````
apache_package_name: httpd
apache_service: httpd
php_package_name: php
````

resto de ficheros así

````
apache_package_name: 'apache2'
apache_service: 'apache2'
php_package_name: 'libapache2-mod-php'
````

encerrar el valor de las valiables con la comilla simple es opcional

el fichero roles/web_servers/tasks/main.yaml cambia a este formato para hacer uso de las variables definidas

````
- name: install apache and php packages
  tags: apache,httpd,php
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}" 
    state: latest
 
- name: start and enable apache service
  tags: apache,httpd
  service:
    name:  "{{ apache_service }}"
    state: started
    enabled: yes
 
- name: change e-mail address for admin
  tags: apache,fedora,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "Fedora"
  register: apache

- name: restart httpd
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: restarted
  when: apache.changed
 
- name: copy html file for site
  tags: apache,apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644

````

uso de los handlers que sirven para disparar el reinicio de un servicio no basándonos en el uso de variables, ya que si tienes que reiniciar varios servicios en un sólo playbook el estado de la variable puede cambiar dependiendo de la última acción realizada.

el playbook roles/web_servers/tasks/main.yaml con la inclusión de un handler para reiniciar el servicio httpd queda así

````
- name: install apache and php packages
  tags: apache,httpd,php
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}" 
    state: latest
 
- name: start and enable apache service
  tags: apache,httpd
  service:
    name:  "{{ apache_service }}"
    state: started
    enabled: yes
 
- name: change e-mail address for admin
  tags: apache,fedora,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "Fedora"
  notify: restart_apache

- name: copy html file for site
  tags: apache,apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644

````

creamos el directorio handlers en el directorio web_servers

````
mkdir ./roles/web_servers/handlers
````

y en dicho directorio creamos el fichero main.yaml

````
vim ./roles/web_servers/handlers/main.yaml
````

con el siguiente contenido

````
- name: restart_apache
  service:
    name: "{{ apache_service }}" 
    state: restarted
````

la etiqueta name debe ser igual al valor que le hemos dado en el playbook a la etiqueta notify, en este caso restart_apache

si cambiamos en el yaml la dirección de correo de contacto, por ejemplo cambiando el .net por .com podemos ver cómo se dispara el handler porque ha habido un cambio en la tarea de web_servers: change email address for admin

````
ansible-playbook site_with_roles.yaml -i inventory_with_groups
````

````
RUNNING HANDLER [web_servers : restart_apache] *************************************************************************************
changed: [ansible@192.168.1.189]

````
# NO HACER NADA DE LO SIGUIENTE PORQUE TOCA EL SSHD_CONFIG Y SE LÍA PARDA PARA LUEGO PODER HACER LOGIN EN LOS SERVERS. NO SE DEBEN HACER GRANDES CAMBIOS EN EL SSHD_CONFIG

uso de templates, en este caso nos va a ser útil suponiendo que cada server tuviese una ssh key diferente para que ansible pueda ejecutar tareas en los targets sin tener que usar el flag de que nos solicite la contraseña de un usuario con permisos de root. Usaremos los templates para poder indicar diferentes ssh keys según el target en cuestión.

````
cat /etc/ssh/sshd_config
````

queremos una copia de ese fichero en git, para ello nos movemos al directorio cd roles/base y creamos ahí el directorio templates

````
cd roles/base
mkdir templates
````


nos cambiamos al directior templates y copiamos el ficher sshd_config

````
cd templates
cp /etc/ssh/sshd_config sshd_config_ubuntu.j2 
````

la extensión .j2 es la extensión por defecto a utilizar en los templates de Ansible (jinja2)

ahora copiemos también el fichero sshd_config del target Fedora, lo hacemos desde la máquina Fedora usando sudo

````
[ansible@fedorasrv01 ssh]$ sudo scp ssh_config fpinero@192.168.1.95:/home/fpinero/ansible_lab/roles/base/templates/sshd_config_fedora.j2
````

editamos los ficheros sshd_config_ubuntu.j2 y sshd_config_fedora.j2 les añadimos

´´´´
AllowUsers {{ ssh_users }}
´´´´

nos movemos al directorio donde tenemos definidas las variables para cada target y les añadimos la definición de la variable

´´´´
cd host_vars
vim *
ssh_users: "ansible, fpinero, simone"   
ssh_template_file: sshd_config_ubuntu.j2 
´´´´
* al fichero de variables de Fedora el nombre del template es sshd_config_fedora.j2

````
ssh_template_file: sshd_config_fedora.j2 
````

también añadimo sel fichero de variables para el equipo desde donde ejecutamos ansible porque tenemos una tarea que instala Terraform en él, este caso es el fpinero@192.168.1.95.yaml que contiene la misma info que cualquiera de los ficheros de variables de un target Ubuntu. <br/>
estos son los ficheros que deben existir en el directorio host_vars

´´´´
ansible@192.168.1.180.yaml
ansible@192.168.1.184.yaml
ansible@192.168.1.189.yaml
ansible@192.168.1.86.yaml
fpinero@192.168.1.95.yaml
´´´´

nos movemos al directorio cd ./roles/base/tasks/ y editamos el main.yaml que tan sólo contiene el usuario y la key a utilizar para conectarnos a los targets sin tener que usar el flag de Ansible para que nos solicite la contraseña de un usuario con permisos de root

````
vim main.yaml
````

y le añadimos una tarea nueva

´´´´
- name: add ssh key for simone
  authorized_key:
   user: simone
   key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAe7/ofWLNBq3+fRn3UmgAizdicLs9vcS4Oj8VSOD1S/ ansible"

- name: generate sshd_config file from temaplate
  tags: ssh
  template: 
    src: "{{ ssh_template_file }}"
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: 0644
  notify: restart_sshd
´´´´

creamos en roles/base en directorio handlers para crear el handler que utilizamos en el main.yaml de base que reinicia sshd

´´´´
mkdir handlers
´´´´

nos movemos dentro del recién creado directorio handlers y creamos el fichero main.yaml

´´´´
- name: restart_sshd
  service:
    name: sshd
    state: restarted
´´´´

´´´´
ansible-playbook site_with_roles.yaml -i inventory_with_groups
´´´´

nuevo playbook site_stress.yaml que instala el paquete stress y ejecuta en todos los targets un stress de la cpu por 10s

````
---
 
- hosts: all
  become: true
  tasks:

    - name: install stress
      package: 
        name: stress

    - name: Execute stress
      command: "stress --cpu 2 --timeout 10s"
````

````
ansible-playbook site_stress.yaml -i inventory_with_groups
````

