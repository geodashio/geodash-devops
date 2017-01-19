# GeoDash DevOps (geodash-devops)

DevOps tools for developing & deploying [GeoDash](http://geodash.io/), including [Ansible](https://www.ansible.com/), [Packer](https://www.packer.io/), [Vagrant](https://www.vagrantup.com/), and [Fabric](http://www.fabfile.org/) configuration files for building and managing Ubuntu and CentOS GeoDash boxes.

# Installation

On the control/host machine, you'll need to install [Ansible](https://www.ansible.com/), [Packer](https://www.packer.io/), [Vagrant](https://www.vagrantup.com/), and [Fabric](http://www.fabfile.org).

**Quick Install**

For some machines, you can quickly install [Ansible](https://www.ansible.com/) and [Fabric](http://www.fabfile.org) with the following:

```
sudo apt-get install python-dev # if not already installed
sudo easy_install pip  # if pip is not already installed
sudo pip install virtualenv virtualenvwrapper
# cd into project directory
sudo pip install -r requirements.txt
```

## Ansible

Ansible is an agent-less provisioning tool for managing the state of machines.  It is used by both [Packer](https://www.packer.io/) and [Vagrant](https://www.vagrantup.com/).  By sharing a common Ansible `playbook` for configuring production machines, building test boxes via Packer, and development boxes for Vagrant, we're able to have dev-prod parity.

To get [Ansible](https://www.ansible.com/) follow the relevant section below.  Also see http://docs.ansible.com/ansible/intro_installation.html#getting-ansible for more information.

#### Mac OS X & Ubuntu

```
sudo easy_install pip  # if pip is not already installed
sudo pip install ansible
```

## Packer

[Packer](https://www.packer.io/) can be used to build virtual machine images.

#### Mac OS X

```
sudo mkdir -p /opt/packer/bin
cd /opt/packer/
sudo wget https://releases.hashicorp.com/packer/0.10.0/packer_0.10.0_darwin_amd64.zip
sudo unzip packer_0.10.0_darwin_amd64.zip
sudo mv packer /opt/packer/bin
cd /usr/local/bin/
sudo ln -s /opt/packer/bin/packer packer
```

#### Ubuntu

```
sudo mkdir -p /opt/packer/bin
cd /opt/packer/
sudo wget 'https://releases.hashicorp.com/packer/0.10.0/packer_0.10.0_linux_amd64.zip'
sudo unzip packer_0.10.0_linux_amd64.zip
sudo mv packer /opt/packer/bin
cd /usr/bin/
sudo ln -s /opt/packer/bin/packer packer
```

## Fabric

[Fabric](http://www.fabfile.org/) provides an easy command line interface for executing remote shell commands and for transferring files between machines.  Fabric is extremely useful for transferring files and managing remote servers.

Follow directions at http://www.fabfile.org/installing.html to install fabric or follow shortcuts below.

#### Mac OS X & Ubuntu

TBD

# Usage

Create a `secret.yml` file in the project root.  Include the `DOMAIN_NAME`, `DB_PASS`, `SUPERUSER_PASSWORD`, and `NPM_PROJECTS`, as applicable, in the `secret.yml` files.  For example:

```
DOMAIN_NAME: geodash.example.com
DB_PASS: geodash
SUPERUSER_PASSWORD: admin
NPM_PROJECTS:
  - ~/geodash.js.git
  - ~/geodash-viewer.git
```

## Vagrant

To add an Ubuntu 14.04 ("Trusty") vagrant box to your control machine, run:

```
vagrant box add ubuntu/trusty64

```

To add the Centos 6.4 vagrant box to your control machine, run:

```
vagrant box add --name "centos/6.4" http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20131103.box

```

To launch the GeoDash virtual machine run:

```
vagrant up
```

To re-provision the machine run:

```
vagrant provision
```

### EC2

First, create a `hosts` file in the project root directory.  The hosts file can be as simple as the public IP address of your ec2 machine.

```
##.##.##.##
```

We'd also suggest hardcoding the ssh_user to override any implict assumptions in playbooks, such as below.

```
##.##.##.## ansible_ssh_user=ubuntu
```

Learn more about managing your inventory at http://docs.ansible.com/ansible/intro_inventory.html.  This hosts file will be referenced in the command below.

Run the following command to provision your ec2 instance.  Be sure to replace `~/osm-slackbot-1.pem` below with the path to your actual key for the machine.

```
# cd into project directory
ansible-playbook --user=ubuntu --connection=ssh --timeout=30 --inventory-file=hosts -v --private-key=~/geodash-demo.pem --extra-vars=@ansible/extra_vars/env/ec2.yml --extra-vars=@ansible/extra_vars/os/ubuntu.yml --extra-vars=@secret.yml ansible/ubuntu_geodash.yml
 ```

## Packer

You can also use [Packer](https://www.packer.io/) to build a virtual machine.

To build a base box for CentOS 6.4, run:

```
packer build -var 'ansible_playbook=ansible/centos_base.yml' -var 'ansible_secret=secret.yml' -var 'ansible_os=centos' packer/centos64.json
```

To build a base box for Ubuntu 14.04, run:

```
packer build -var 'ansible_playbook=ansible/ubuntu_base.yml' -var 'ansible_secret=secret.yml' -var 'ansible_os=ubuntu' packer/ubuntu1404.json
```

#### Adding Your New Box

After you created your box, you can add with:

```
vagrant box add --name "geodash_base" packer_virtualbox-ovf_virtualbox.box
```

Add the box to the [Vagrantfile](https://github.com/geodashio/geodash-devops/blob/master/Vagrantfile) to provision with it.

## Launch

Once the image is provisioned, ssh into the machine via:

```
# cd into geodash-devops.git directory
vagrant ssh
```

Once in the virtual machine, run:

```
workon geodash
cd geodash-server.git
paver stop
paver reset_hard
paver setup
paver start -b 0.0.0.0:8000  # Launches Django and GeoServer.  Listens to all addresses on port 8000.
```

```
gunicorn geodashserver.wsgi --check-config
```

## Fabric

TBD


# GeoDash Developer Workflow

Below are some normative suggestions on how to setup an effective developer workflow for GeoDash using this repo.

First, you should share key repos from the `host` into the `guest`.  Uncomment the following lines in your `Vagrantfile`.  Preserve the `.git` suffix for the folders, since it makes it mush easer to identify them as versioned repos.

```
config.vm.synced_folder "~/workspaces/public/geodash-viewer.git", "/home/vagrant/geodash-viewer.git"
config.vm.synced_folder "~/workspaces/public/geodash-server.git", "/home/vagrant/geodash-server.git"
config.vm.synced_folder "~/workspaces/public/geodash-base.git", "/home/vagrant/geodash-base.git"
```

Second, create 1 line chained commands for building library code and copying over into implementing applications.  For example, the first line below builds GeoDashJS and then copies into `geodash-base`, which is used by `geodash-viewer`.  The second line builds `geodash-viewer`.


```
cd ~/geodash.js.git; npm run build; cp dist/* ~/geodash-base.git/lib/geodashjs/0.0.1/
cd ~/geodash-viewer.git; gulp
```
