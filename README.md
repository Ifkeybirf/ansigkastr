# Ansible GK Astra test task

## Preparation
- obtain Linux host machine (I'd prefer Debian 11)
- install ansible (version 2.10.8 available out of the box from system repo)
```sh
   apt install -y ansible
```
- install vagrant with command
 ```sh
    apt install -y vagrant (2.2.14 available from system repo out of the box)
 ```
- install Virtualbox 6.1 from official site virtualbox.org (latest vagrant from system repo does not work with virtualbox > 6.1)


due to sanctions vagrant box automatic installation does not available by default in RU, so

- we need to have download box image from the https://app.vagrantup.com/generic/boxes/ubuntu2004 for appropriate virtualization backend
and import it manually with command like below
```sh
vagrant box add full_path_to_downloaded_skeleton --name _image_name_ (image_name = "generic/ubuntu2004" based on tech spec in our case)
```

- to change default box disk size plugin vagrant-disksize need to be installed (but it is not mandatory):
```sh
vagrant plugin install vagrant-disksize
```
Vagrant:17 line can be uncommented in case it is installed



you probably need to install ansible modules that are not installed in your system by default:
```sh
ansible-galaxy collection install -r requirements.yml
```

there is already generated inventory file, but in case hosts.txt was modified
you have to execute inventory re-generation command:
```sh
ansible-inventory -i hosts.txt --list --yaml --export --output=inventory
```


