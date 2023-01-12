# Ansible GK Astra test task

## Preequisites
- Modern Linux host machine (I'd prefer Debian 11)
- ansible (version 2.10.8 available out of the box from system repo on Debian 11)
```sh
   apt install -y ansible
```

- install vagrant with command
 ```sh
    apt install -y vagrant
 ```
version 2.2.14 available from system repo out of the box on Debian 11

- install Virtualbox 6.1 from official site virtualbox.org (latest vagrant from Debian 11 system repo does not work with virtualbox > 6.1)

- downloaded _vagrant box_
due to sanctions _vagrant box_ automatic installation does not available by default in RU, so we need to have download box image from the https://app.vagrantup.com/generic/boxes/ubuntu2004 for appropriate virtualization backend and import it manually with command like below
```sh
vagrant box add full_path_to_downloaded_skeleton --name _image_name_
```
where image_name = "generic/ubuntu2004" based on tech spec in our case


- to change default box disk size plugin vagrant-disksize need to be installed (but it is not mandatory):
```sh
vagrant plugin install vagrant-disksize
```
Vagrant:17 line can be uncommented in case it is installed


### Ansible modules
you probably need to install ansible modules that are not installed in your system by default:
```sh
ansible-galaxy collection install -r requirements.yml
```
### Inventory
there is already generated inventory file, but in case hosts.txt was modified
you have to execute inventory re-generation command:
```sh
ansible-inventory -i hosts.txt --list --yaml --export --output=inventory
```
## Results
by command `vagrant up` actions below will be done:
- VM Ubuntu 20.04.5 LTS will be provisioned
- ports below will be exposed from VM to host machine
- - 20422 - SSH entry point to provisioned VM
- - 3000 - Grafana UI
- - 9090 - Prometheus entry point (needed for debug only, in general works without this)
- - 9100 - node_exporter daemon, which collects metrics from VM and provide them to Prometheus

after VM provisioned there are 2 docker containers executed inside: with Prometheus and Grafana
- Grafana credentials is admin:123qwe
- by default [Dashboard #1860](https://grafana.com/grafana/dashboards/1860) opened
- unauthorized access to the default dashboard allowed
- grafana logs are available at `/docker/grafana/grafana_logs/` inside VM


## Known Problems / Issues / Limitations
- VM Ubuntu 20.04.5 uses NAT network and ansible connects to localhost. So there may be varning that `REMOTE HOST IDENTIFICATION HAS CHANGED`.
in this case you have to remove localhost record(s) from your ~/.ssh/known_hosts file.
can be avoided by change ansible config in a way below:
```
$ cat ~/.ansible.cfg
[defaults]
deprecation_warnings=false
ansible_deprecation_warnings=false

[ssh_connection]
ssh_args= -o GlobalKnownHostsFile=/dev/null -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no
```
it also added to the progect's ansible config, but it may be prohibited for using due to security policies.

- There are no templates were used.
more correct way is to put template for `node_exporter system.d config`, `docker-compose` files
and not to hardcode some options (like grafana admin password)
but I was out of time by making it work in general, so i did not make it beauty.

- There is an ansible module `community.grafana` for manage Grafana that has **not** been used:
-- it warn that it is not compatible with ansible 2.10.8
-- it is not compatible with fresh Grafana versions
-- it looks like does not work with dockered Grafana (but i did not investigate it deeply)

- General problem that I was not able to make this dashboard work in general (mean draw graphics) 🤔
I see that 
-- node_exporter collects data from VM node (it can be seen by opening http://localhost:9100)
-- Prometheus is able to fetch data from node_exporter (http://localhost:9090)
-- Grafana is able to use Prometheus as default data source
-- Dashboards from Prometheus can be imported to grafana (checked through UI), and they show some data
but [Dashboard #1860](https://grafana.com/grafana/dashboards/1860) does not show any collected data.
I did not found why.
I've tried to install Grafana right inside VM from .deb package and import dashboard by it's # through Web UI - but it does not work that way either

