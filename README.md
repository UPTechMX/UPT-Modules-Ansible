# UPT-Modules-Ansible
[Ansible](https://docs.ansible.com) playbooks for automatic deployment of the UrbanPerformance and Distance Evaluation modules for the Urban Planning Tools (UPT), comprising of the following repositories:
* [UPT-UrbanPerformance](https://github.com/UPTechMX/UPT-UrbanPerformance), a Python/Django/Celery-based module for the evaluations done for UrbanPerformance.
* [UPT-Distance-Module](https://github.com/UPTechMX/UPT-Distance-Module), a Python/Django/Celery-based module for distance evaluations of layers uploaded through Oskari.

## Requirements
### Target server
* A fresh installed [Ubuntu Server 18.04](https://ubuntu.com/download/server) (minimum 8 GB RAM and 30 GB of free space) with a user that is part of the `sudo` group. Below is an example of a user called `username` that is created and added to the `sudo` group. 
  ```
  adduser username
  usermod -aG sudo username
  ```
* The server is fully updated
  ```
  sudo apt update && sudo apt upgrade
  ```
* Ansible installed on the target server
  ```
  sudo apt update
  sudo apt install -y software-properties-common
  sudo apt-add-repository --yes --update ppa:ansible/ansible
  sudo apt install -y ansible
  ansible --version
    ansible 2.9.6
      config file = /etc/ansible/ansible.cfg
      configured module search path = [u'/home/user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
      ansible python module location = /usr/lib/python2.7/dist-packages/ansible
      executable location = /usr/bin/ansible
      python version = 2.7.17 (default, Nov  7 2019, 10:07:09) [GCC 7.4.0]
  ```
  
## How to run
SSH to the target server and perform the following steps:
* Clone this repo (install `git` package if necessary)
  ```
  # clone the ansible playbook repo to /opt
  cd /opt
  # become root user
  sudo su
  git clone https://github.com/UPTechMX/UPT-Modules-Ansible.git
  cd UPT-Modules-Ansible
  ```
* Adjust `install_upt_modules.yml`
```
#################
- hosts: all
  vars:
    user: changeme # <------ Change this variable to user belonging to sudo group
#################
```
* Adjust ports for the modules to listen to:
  * ```UPT-Distance.j2```
  ```
  # Edit file for distance evaluation module
  nano upt_modules/UPT-Distance.j2
  
  ##########################
  #!/bin/bash

  function django(){
          echo "Starting UPT Distance"
          cd /home/{{ user }}/upt-distance/
          source /home/{{ user }}/python3/bin/activate && python manage.py runserver 0.0.0.0:90 # <------ Change '90' to a different port if needed
  }
  django
  ##########################
  ```
  * ```UPT-UP.j2```
  ```
  # Edit file for UrbanPerformance module
  nano upt_modules/UPT-UP.j2
  
  ##########################
  #!/bin/bash

  function django(){
          echo "Starting UPT UP"
          cd /home/{{ user }}/upt-up/
          source /home/{{ user }}/python3/bin/activate && python manage.py runserver 0.0.0.0:91 # <------ Change '91' to a different port if needed
  }
  django
  ##########################
  ```
* Run the playbook
    * Recommended method: run the install all playbook to install everything in order
        ```
        cd /opt/UPT-Modules-Ansible
        # become root user
        sudo su
        ansible-playbook -K -i inventory install_upt_modules.yml
          BECOME password: enter the user's sudo password
        ```
* Reboot the server
  ```
  sudo reboot
  ```
* Once the playbook has finished running, the following should be true:
  * `upt-up` module is listening to the appropriate port
  * `upt-distance` module is listening to the appropriate port
  * `upt-celery-distance` service is enabled and running
  * `upt-celery-up` service is enabled and running
* Done!
