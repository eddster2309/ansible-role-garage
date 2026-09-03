# Ansible Role for Garage
This is an ansible role to install and configure [Garage](https://garagehq.deuxfleurs.fr/) an open source object storage service.

I initially built this role for Fedora 41 but it should be distro agnostic.

Currently this playbook can do the following:
- Install Garage
- Automatically connect and configure Garage nodes
- Install Nginx as a load balancer between nodes
- Install Keepalived to create a VIP between nodes. Using [evrardjp.keepalived](https://github.com/evrardjp/ansible-keepalived) role.
- Configure the firewall on systems using firewalld to allow for secure comunications between nodes
- Create buckets and configure relavent settings (quotas and web access).
- Create access keys and assign defined access

## Installing this role
This role is published on Ansible Galaxy:
```
ansible-galaxy install eddster2309.garage
```

## Configuring this role
The configuration options for this role can be found in [defaults/main.yml](./defaults/main.yml). **Make sure to change all secrets!**

### Multiple data volumes
Garage's data storage can be spread across multiple volumes (e.g. separate disks or mounts) by defining several entries in `garage_data_directories`:
```yaml
garage_data_directories:
  - path: /mnt/disk1/garage-data
    capacity: 2T
  - path: /mnt/disk2/garage-data
    capacity: 2T
  - path: /mnt/disk3/garage-data
    read_only: true
```
Each volume needs a `path`, and a `capacity` unless `read_only` is set to `true`. Refer to [Garage's documentation](https://garagehq.deuxfleurs.fr/documentation/reference-manual/configuration/#data_dir) for more details. The sum of volume capacities should generally match `garage_node_capacity`, which is what's announced to the cluster layout.

## Upgrading Garage
By default this role will always pull the latest version of the binary, but it is reccomended that you set the `garage_version` variable to a particular version (not `latest`) and then read the [release notes](https://git.deuxfleurs.fr/Deuxfleurs/garage/releases) before upgrading.

## Managing buckets and keys
This role can create and manage keys and buckets withing Garage. It will not delete buckets or keys.
### Creating buckets
Buckets can ne defined within the `garage_buckets` variable as found in [defaults/main.yml](./defaults/main.yml). The only required field is the name, by default web access will be disabled and there will be an unlimited quota. Bucket name must be unique.
### Creating keys
Keys can be defined within the `garage_keys` variable as found in [defaults/main.yml](./defaults/main.yml). The ID must be unique and me a 24 character hexadecimal string prefixed with `GK`. The secret must be a 64 character hexadecimal string.

To generate a key that complies to all the requirements use the `keygen.sh` bash script included with this role.

## Ngnix Deployment
The nginx deployment that this role deploys is meant to load balance and provide subdomain support for the S3 and web endpoints on a single port. If you have another reverse proxy I would recomend you set up your own loadbalancer in accordance with [Garage's docs](https://garagehq.deuxfleurs.fr/documentation/cookbook/reverse-proxy/).

### SSL Support
This role supports enabling SSL with pre-existing certificates, this is due to the complexity of all the different ways certs can be generated, but if you want to use Lets Encrypt (certbot), i would reccomend using [Jeff Geerlings Certbot Ansible role](https://github.com/geerlingguy/ansible-role-certbot).
