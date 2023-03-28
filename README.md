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

