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
ansible-playbook --ask-become-pass revome_apache.yaml
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



