# Keep Alive config

## Building the VMs

  1. Download and install [VirtualBox](https://www.virtualbox.org/wiki/Downloads).
  2. Download and install [Vagrant](http://www.vagrantup.com/downloads.html).
  3. [Mac/Linux only] Install [Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html).
   

### Variables and hosts
All host and variables for hosts are located in the **inventory** file



### Hosts
```
nginx1 - 192.168.28.2
nginx2 - 192.168.28.3
nginx3 - 192.168.28.4
```

### Virtual IP
```
192.168.28.50
```

### Ansible Roles 
* firewall
* nginx 
* keepalived
* haproxy

### Diagram
```
                                                          ----------------------------
                                           /------------->|nginx1-192.168.28.50:80   |
                                          /               |haproxy:192.168.28.50:8080|
					 /		  ----------------------------
        |----                           /
        |   |  192.168.29.50           /                
        |   |  nginx port : 80        /                  -----------------------------
--------|VIP|------------------------------------------->|nginx2-192.168.28.50:80    |                
        |   |  192.168.29.50          \                  |haproxy2-192.168.28.50:8080|
        |   |  haproxy port : 8080     \		 |----------------------------
        |----                           \
                                         \               |-----------------------------
                                          \	         |nginx2-192.168.28.50:80     |
                                           \-------------|haproxy2-192.168.28.50:8080 |
                                                         |----------------------------|
```
### Starting the stack
```
vagrant up
```

### Testing nginx
Keep alived works in Active / Passive mode in MASTER mode so it will show only the resule from only on nginx.

Test nginx on virtual IP
```
for i in {1..5} ; do curl http://192.168.29.50/;done
nginx3
nginx3
nginx3
nginx3
nginx3
```
Stopping the VM Nginx3 network
```
vagrant ssh nginx3
sudo ifconfig eth1 down
```
Test nginx on virtual IP
```
for i in {1..5} ; do curl http://192.168.29.50/;done
nginx1
nginx1
nginx1
nginx1
nginx1
```
Start the VM network again
```
vagrant ssh nginx3
sudo ifconfig eth1 up
```
Test nginx on virtual IP again
```
curl http://192.168.29.50/
nginx3
```
### Testing haproxy
```
for i in {1..5} ; do curl http://192.168.29.50:8080/;done
nginx3
nginx1
nginx2
nginx3
nginx1
```
**It show all 3 nginx VM's**


### Stop haproxy on Virtual IP address 
```
vagrant ssh nginx3
sudo ifconfig eth1 down
```
Test haproxy 
```
for i in {1..5} ; do curl http://192.168.29.50:8080/;done
nginx2
nginx1
nginx2
nginx1
nginx2
```
**It shows only 2 nginx VM's**
