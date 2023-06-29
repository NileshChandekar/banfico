```
Env Details:
```

|Base OS Machine|Kernel|
|----|----|
|Ubuntu 18.04.6|4.15.0-208-generic|


```
PART A
```

```
Scenerio:- Considering VirtualBox and Vagrant is not installed
```


* In this bash script, the ``get_linux_distribution`` function is used to identify the Linux distribution. 

* It checks various common files ``(/etc/os-release, /etc/lsb-release, /etc/debian_version, /etc/redhat-release)`` to determine the distribution name. 

* If no specific distribution is found, it defaults to the output of ``uname -s``.

* The ``install_virtualbox`` function checks the distribution name and installs VirtualBox accordingly. 

* If the distribution is ``Ubuntu``, it uses ``apt-get`` to update the package lists and install ``VirtualBox``. 

* If the distribution is ``CentOS or Red Hat``, it uses ``yum`` to install ``VirtualBox``.

* The ``show_service_status`` function checks the distribution name and shows the status of the VirtualBox service. 

* The ``install_vagrant`` function checks the distribution name and installs Vagrant accordingly. 

* If the distribution is ``Ubuntu``, it uses ``apt-get`` to update the package lists and install ``Vagrant``. 

* If the distribution is ``CentOS or Red Hat``, it uses ``yum`` to install ``Vagrant``.


```
root@saif-osa-bionic-deploy:/tmp/banfico# cat vagrant.sh 
#!/bin/bash

get_linux_distribution() {
    lsb_dist=""
    if [ -r /etc/os-release ]; then
        lsb_dist="$(. /etc/os-release && echo "$ID")"
    elif [ -r /etc/lsb-release ]; then
        lsb_dist="$(. /etc/lsb-release && echo "$DISTRIB_ID")"
    elif [ -r /etc/debian_version ]; then
        lsb_dist="debian"
    elif [ -r /etc/redhat-release ]; then
        lsb_dist="$(cat /etc/redhat-release | awk '{print $1}')"
    else
        lsb_dist="$(uname -s)"
    fi
    echo "$lsb_dist"
}

install_virtualbox() {
    dist_name=$(get_linux_distribution)

    if [[ "$dist_name" == "ubuntu" ]]; then
        apt-get update
        apt-get install -y virtualbox
    elif [[ "$dist_name" == "centos" || "$dist_name" == "redhat" ]]; then
        yum install -y virtualbox
    else
        echo "Unsupported distribution. Cannot install VirtualBox."
        exit 1
    fi
}

show_service_status() {
    dist_name=$(get_linux_distribution)

    if [[ "$dist_name" == "ubuntu" ]]; then
        service virtualbox status
    elif [[ "$dist_name" == "centos" || "$dist_name" == "redhat" ]]; then
        systemctl status virtualbox
    fi
}

install_vagrant() {
    dist_name=$(get_linux_distribution)

    if [[ "$dist_name" == "ubuntu" ]]; then
        apt-get update
        apt-get install -y vagrant
    elif [[ "$dist_name" == "centos" || "$dist_name" == "redhat" ]]; then
        yum install -y vagrant
    else
        echo "Unsupported distribution. Cannot install Vagrant."
        exit 1
    fi
}

# Identify Linux distribution
dist_name=$(get_linux_distribution)
echo "Distribution: $dist_name"

# Install VirtualBox
install_virtualbox

# Show service status
show_service_status

# Install Vagrant
install_vagrant

root@saif-osa-bionic-deploy:/tmp/banfico# 
```

* run the scipt 

```
bash -x vagrant.sh
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# systemctl status virtualbox.service
● virtualbox.service - LSB: VirtualBox Linux kernel module
   Loaded: loaded (/etc/init.d/virtualbox; generated)
   Active: active (exited) since Thu 2023-06-29 08:27:02 UTC; 4h 0min ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 0 (limit: 4915)
   CGroup: /system.slice/virtualbox.service

Jun 29 08:27:02 saif-osa-bionic-deploy systemd[1]: Starting LSB: VirtualBox Linux kernel module...
Jun 29 08:27:02 saif-osa-bionic-deploy virtualbox[4096984]:  * Loading VirtualBox kernel modules...
Jun 29 08:27:02 saif-osa-bionic-deploy virtualbox[4096984]:    ...done.
Jun 29 08:27:02 saif-osa-bionic-deploy systemd[1]: Started LSB: VirtualBox Linux kernel module.
root@saif-osa-bionic-deploy:/tmp/banfico# 
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# vagrant -v
Vagrant 2.3.7
root@saif-osa-bionic-deploy:/tmp/banfico# 
```

```
PART B
```

```
Scenerio 2: Provision 2 small CentOS instances (256MB RAM, 1 CPU) of HAProxy using Vagrant script & using provider as VirtualBox
```

* In the above script, we define two virtual machines ("haproxy1" and "haproxy2") using the CentOS 7 box. Each virtual machine is configured with 256MB RAM and 1 CPU core.


```
root@saif-osa-bionic-deploy:/tmp/banfico# cat Vagrantfile 
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "256"
    vb.cpus = 1
  end

  config.vm.define "haproxy1" do |node|
    node.vm.hostname = "haproxy1"
    node.vm.network "private_network", ip: "192.168.33.10"
    node.vm.provision "shell", inline: <<-SHELL
      # Provisioning commands for HAProxy server 1
    SHELL
  end

  config.vm.define "haproxy2" do |node|
    node.vm.hostname = "haproxy2"
    node.vm.network "private_network", ip: "192.168.33.11"
    node.vm.provision "shell", inline: <<-SHELL
      # Provisioning commands for HAProxy server 2
    SHELL
  end
end
root@saif-osa-bionic-deploy:/tmp/banfico# 
```

```
vagrant up
```

* After sometime you will have two machine ready with you, if all goes well. 

```
root@saif-osa-bionic-deploy:/tmp/banfico# vagrant status
Current machine states:

haproxy1                  running (virtualbox)
haproxy2                  running (virtualbox)

root@saif-osa-bionic-deploy:/tmp/banfico# 
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# vagrant ssh haproxy1
Last login: Thu Jun 29 12:16:47 2023 from 10.0.2.2
[vagrant@haproxy1 ~]$ uptime
 12:33:21 up 23 min,  1 user,  load average: 0.00, 0.12, 0.65
[vagrant@haproxy1 ~]$ 
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# vagrant ssh haproxy2
[vagrant@haproxy2 ~]$ uptime
 12:33:43 up 18 min,  1 user,  load average: 0.00, 0.01, 0.06
[vagrant@haproxy2 ~]$ 
```

* Install ansible 

```
apt install ansible -y 
```

* Inventory file 

```
root@saif-osa-bionic-deploy:/tmp/banfico# cat ip.txt 
192.168.33.10
192.168.33.11
root@saif-osa-bionic-deploy:/tmp/banfico# 
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# ansible -i /tmp/banfico/ip.txt -m ping all
192.168.33.11 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.33.10 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
root@saif-osa-bionic-deploy:/tmp/banfico# 
```


```
ansible -i /tmp/banfico/ip.txt all -m file -a "path=~/.ssh state=directory mode=0700"
```

```
ansible -i /tmp/banfico/ip.txt all -m copy -a "src=~/.ssh/authorized_keys dest=~/.ssh/authorized_keys mode=0600"
```

* Install haproxy and keepalived 

```
root@saif-osa-bionic-deploy:/tmp/banfico# cat install_haproxy_keepalived.yaml
---
- name: Configure Keepalived and HAProxy
  hosts: all
  become: true
  vars:
    haproxy_primary_ip: 192.168.33.10
    haproxy_backup_ip: 192.168.33.11
    virtual_ip: 192.168.33.199
    haproxy_backend_servers:
      - name: haproxy1
        ip: 192.168.33.10
      - name: haproxy2
        ip: 192.168.33.11

  tasks:
    - name: Install Keepalived and HAProxy
      yum:
        name: "{{ item }}"
        state: present
      loop:
        - keepalived
        - haproxy

    - name: Configure Keepalived
      template:
        src: /tmp/banfico/keepalived.conf
        dest: /etc/keepalived/keepalived.conf
        owner: root
        group: root
        mode: '0644'
      notify:
        - restart keepalived

    - name: Configure HAProxy
      template:
        src: /tmp/banfico/haproxy.cfg
        dest: /etc/haproxy/haproxy.cfg
        owner: root
        group: root
        mode: '0644'
      notify:
        - restart haproxy

  handlers:
    - name: restart keepalived
      service:
        name: keepalived
        state: restarted

    - name: restart haproxy
      service:
        name: haproxy
        state: restarted

root@saif-osa-bionic-deploy:/tmp/banfico# 
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# cat keepalived.conf
global_defs {
    router_id HAProxy1
}

vrrp_script chk_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight -2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 50
    priority 100
    advert_int 1
    authentication {
        auth_type admin
        auth_pass admin
    }
    virtual_ipaddress {
        192.168.33.199
    }
    track_script {
        chk_haproxy
    }
}

virtual_server 192.168.33.199 80 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.33.10 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }

    real_server 192.168.33.11 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}


root@saif-osa-bionic-deploy:/tmp/banfico# 
```

```
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend http_front
    bind *:80
    bind *:443 ssl crt /etc/haproxy/certificate.pem
    mode http
    option httplog
    default_backend http_back

backend http_back
    balance roundrobin
    server google_server www.google.com:443 check ssl verify none
    server google_server www.google.com:80 
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# ansible-playbook -i ip.txt install_haproxy_keepalived.yaml 
```


```
root@saif-osa-bionic-deploy:/tmp/banfico# cat create_haproxy_cert2.yml
#!/bin/bash

# Generate private key
openssl genpkey -algorithm RSA -out /tmp/banfico/private.key

# Generate certificate signing request (CSR)
openssl req -new -key /tmp/banfico/private.key -out /tmp/banfico/certificate.csr \
    -subj "/C=US/ST=California/L=San Francisco/O=Example Inc./OU=IT Department/emailAddress=admin@example.com/CN=example.com"

# Generate self-signed certificate
openssl x509 -req -days 365 -in /tmp/banfico/certificate.csr -signkey /tmp/banfico/private.key -out /tmp/banfico/certificate.crt

# Combine private key and certificate into PEM file
cat /tmp/banfico/private.key /tmp/banfico/certificate.crt > /tmp/banfico/certificate.pem

# Set proper permissions for certificate and key
chown haproxy:haproxy /tmp/banfico/private.key /tmp/banfico/certificate.*
chmod 600 /tmp/banfico/private.key /tmp/banfico/certificate.*


ansible -i /tmp/banfico/ip.txt all -m copy -a "src=/tmp/banfico/private.key dest=/etc/haproxy/"
ansible -i /tmp/banfico/ip.txt all -m copy -a "src=/tmp/banfico/certificate.crt dest=/etc/haproxy/"
ansible -i /tmp/banfico/ip.txt all -m copy -a "src=/tmp/banfico/certificate.csr dest=/etc/haproxy/"
ansible -i /tmp/banfico/ip.txt all -m copy -a "src=/tmp/banfico/certificate.pem dest=/etc/haproxy/"
ansible -i /tmp/banfico/ip.txt all -m shell -a "systemctl restart haproxy"
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# cat /etc/hosts | tail -n1 
192.168.33.199 my-google.com
root@saif-osa-bionic-deploy:/tmp/banfico# 
```

```
root@saif-osa-bionic-deploy:/tmp/banfico# curl --insecure https://my-google.com | grep -i href
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1561  100  1561    0     0  10619      0 --:--:-- --:--:-- --:--:-- 10619
  <a href=//www.google.com/><span id=logo aria-label=Google></span></a>
root@saif-osa-bionic-deploy:/tmp/banfico# 
```


