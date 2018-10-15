# Ansible playbooks for Laravel applications

* PHP
* nginx + php-fpm with multiple virtual hosts
* SSL certificates and renewal with Let's Encrypt
* composer
* redis
* MySQL
* Crontab entries for queue workers and scheduled tasks

## Getting started

### Supported distributions

* Ubuntu Trusty
* Debian (Not tested)

Other versions of the distributions listed above might work, but no guarantees given. See testing if you want to try running the distribution of your choice in virtual machines.

### Installing Ansible
You can install Ansible by following the [official installation documentation](http://docs.ansible.com/ansible/intro_installation.html). For Windows 10 users, you can use the instructions for Ubuntu after [installing the Linux subsystem](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide). For older Windows version, please [check this tutorial on running Ansible on cygwin](https://www.jeffgeerling.com/project/running-ansible-within-windows).

### Clone the repository
```shell
git clone git@github.com:Rishats/ansible_laravel.git
```

### Configure your inventory files
The inventory files are located in the inventory directory. Check the example for development and read the [Ansible documentation](http://docs.ansible.com/ansible/intro_inventory.html).

### Configure your host variables
Host variables control some of the aspects how the machine is configured. Store the host variable file with the same name as the hostname in the inventory file. Below is an example for the local development machine. 

```yaml
# Let's Encrypt settings
enable_ssl: false
letsencrypt_email: rihasultanov@gmail.com

# Extra PHP packages
php_extra_packages:
    - php7.2-soap
    - php7.2-intl

# PHP memory limit
php_memory_limit: "1024M"

# A list of virtual hosts.
virtualhosts:
  project:
    servernames:
    - demo.rishat-sultanov.ru

# Auth for testing and staging.
htpasswd_credentials:
  login: developer
  password: developer

# MySQL
mysql_root_password: super-secure-password
mysql_databases:
  - name: projectname
    encoding: utf8
    collation: utf8_general_ci
  - name: testing_projectname
    encoding: utf8
    collation: utf8_general_ci
mysql_users:
  - name: projectname
    host: "%"
    password: projectname_pwd
    priv: "projectname.*:ALL"
  - name: testing_projectname
    host: "%"
    password: testing_projectname_pwd
    priv: "testing_projectname.*:ALL"

# User for VPS
system_user:
  user_login: projectname
  user_password: projectname_pwd
```

### Securing your host variables
Since you storing highly confidential information like production database passwords in the host variables and .env templates it is highly recommend that encrypt them with a symmetric AES key. Luckily Ansible has built-in tool for this called Vault. Please [read the Vault documentation](http://docs.ansible.com/ansible/playbooks_vault.html) on how to set it up.

#### Supported variables

| Name                                   | Description                                                                                                            | Required |
|----------------------------------------|------------------------------------------------------------------------------------------------------------------------|----------|
| enable_ssl                             | Controls whether the Playbook configures Let's Encrypt certificates on all the virtual hosts.                          | No       |
| letsencrypt_email                      | Email address for sending the expiry notices for the certificates.                                                     | No       |
| php_extra_packages                     | List of extra php-packages you want to install                                                                         | No       |
| php_memory_limit                       | Sets the memory_limit in php.ini                                                                                       | No       |
| virtualhosts.name                      | Shortname used for folders like /var/www/name/public                                                                   | Yes      |
| virtualhosts.name.servernames          | List of hostnames for the virtual host. Must be a valid FQDN if you set enable_ssl to true.                            | Yes      |
| mysql_root_password                    | Root password for MySQL                                                                                                | Yes      |
| mysql_databases.name                   | Name of MySQL database that will generated                                                                             | Yes      |
| mysql_databases.name.mysql_user        | Name of normal MySQL user for that database. Place this in your .env file.                                             | Yes      |
| mysql_databases.name.mysql_password    | Password for the MySQL user. Place this in your .env file.                                                             | Yes      |                                   | Yes       |     |

## Running

### Checking SSH access
After you have done with the configuration make sure you have SSH access to the servers that listed on the inventory file. You can test this by using the ping module. Troubleshooting the SSH connection is the out of scope of this repository. Please note that you can set the SSH user and port in the inventory file, see the development for example.

```shell
ansible -i inventory/development webservers -m ping
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Running the Playbook
You can run the playbook on all hosts with the following command:
```shell
ansible-playbook -i inventory site.yml
```
Provisioning takes a while for the first time so go grab a coffee. If there were no problems, you should have readily provisioned machine and you can deploy your Laravel application to the /var/www/`virtualhosts.name` folder.  

### Running custom playbooks
Sometimes you have something specific which Barn does not cover. In that case you can create your custom playbooks. Store the playbooks to `roles/custom/files` with the .yml extension and they will be executed on all roles.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
