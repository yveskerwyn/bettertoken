# Getting started with Digital Me

## Setup

![](https://docs.google.com/drawings/d/e/2PACX-1vRV2F6GTuTQBR1_48Xf8-UrMM1Dhn89zETs4qp29dxkXNqG9tcSG9243Qn6LvrM1hPxhnakDkzh_rDj/pub?w=960&h=720)


## Steps

- [Create the ZeroTier private network](#create-zt-priv-network)
- [Join the private ZeroTier network](#join-private-network)
- [Join the public ZeroTier farmer network](#join-farmer-network)
- [Start a local Digital Me Robot](#dm-robot)
- [Access the Node-Robot of your ThreeFold node](#node-robot)
- [Stream the Node-Robot output of your ThreeFold node](#stream)
- [Reset Node Robot (if needed)](#reset-node-robot)
- [Delete VMs using the Zero-OS client (if needed)](#delete-vms)
- [Delete containers using the Zero-OS client (if needed)](#delete-containers)
- [Hard reset your ThreeFold node (if needed)](#hard-reset)
- [Create a ZeroTier client service](#create-client-service)
- [Create the VM machine](#create-vm)
- [Create a virtual Web app](#create-webapp)
- [Create a public gateway](#public-gw)


## Create the ZeroTier private network

Install ZT if needed:
```python
j.tools.prefab.local.network.zerotier.install()
j.tools.prefab.local.network.zerotier.start()
```

List the configuration instances for your ZeroTier accounts:
```python
j.clients.zerotier.list()
```

Set the name of the ZeroTier configuration instance for the ZeroTier account you want to create or use:
```python
zt_config_instance_name = 'my_zt_account'
```

In case you have already created a configuration instance for your ZeroTier account just get it:
```python
zt_client = j.clients.zerotier.get(instance=zt_config_instance_name)
```

Optionally, in order to delete your existing ZeroTier configuration instance:
```python
j.clients.zerotier.delete(instance=zt_config_instance_name)
```

In case you need to create a (new) JumpScale client for ZeroTier:
```python
zt_token = '***'
zt_cfg = dict([('token_', zt_token)])
zt_client = j.clients.zerotier.get(instance=zt_config_instance_name , data=zt_cfg)
```

In order to list all your available ZeroTier networks:
```python
zt_client.networks_list()
```

If this ZeroTier network was already created before, get it using the network id:
```python
zt_private_network_id = 'e4da7455b248690c'
zt_private_network = zt_client.network_get(network_id=zt_private_network_id)
```

Optionally, in case you want to start from scratch, and delete the previously created private network:
```python
zt_private_network_id = 'e4da7455b248690c'
zt_client.network_delete(network_id=zt_private_network_id)
```

If not created yet the network, create it:
```python
auto_assign_range = '10.147.20.0/24'
zt_private_network_name = 'private_network'
zt_private_network = zt_client.network_create(public=False, name=zt_private_network_name, auto_assign=True, subnet=auto_assign_range)
zt_private_network_id = zt_private_network.id
```

Check the result:
```python
#zt_private_network = zt_client.network_get(network_id=zt_private_network_id)
zt_private_network.config
```

<a id="join-private-network"></a>

## Join the private ZeroTier network

```python
j.tools.prefab.local.network.zerotier.network_join(network_id=zt_private_network_id, zerotier_client=zt_client)
```

Optionally, get the IP address assigned to you in this private ZeroTier network:
```python
zt_machine_addr = j.tools.prefab.local.network.zerotier.get_zerotier_machine_address()
zt_private_network_member = zt_private_network.member_get(address=zt_machine_addr)
zt_private_network_member_ip_addr = zt_private_network_member.get_private_ip()
```

<a id="join-farmer-network"></a>

## Join the public ZeroTier farmer network

```python
zt_public_farmer_network_id = 'c7c8172af1f387a6'
j.tools.prefab.local.network.zerotier.network_join(network_id=zt_public_farmer_network_id)
```

Optionally, if you have the rights, get more information about this network:
```python
zt_public_farmer_network = zt_client.network_get(network_id=zt_public_farmer_network_id)
```

In the browser you can check the same information, but again, only if you have the appropriate rights:
https://my.zerotier.com/api/network/c7c8172af1f387a6


<a id='dm-robot'></a>

## Start a local Digital Me Robot

See: https://github.com/Jumpscale/digital_me/tree/master/specs

Digital Me is actually a set of service templates for getting things done in the ThreeFold Grid.

In order to use these Digital Me service templates you need a Zero-Robot, which can be seen as your Digital Me.

So first step is to get a running instance of a Zero-Robot on your local machine, or any (virtual) machine you have admin access to.

@todo, also discuss the Docker option, and make sure demo can be done from Windows machine.

Based on the steps as documented in [Getting started with Zero-Robot](https://github.com/yveskerwyn/jumpscale/blob/master/12-zero-robot.md) do the following.

First make sure you have an up to date JumpScale installation, execute the following in the `core9`, `lib9` and `prefab9` directories:
```bash
git pull
pip install -e .
```

Then install 0-robot using the [0-robot installation instructions](https://github.com/zero-os/0-robot/blob/development/docs/getting_started.md#install-0-robot):
```bash
apt-get install -y libsqlite3-dev
mkdir -p /opt/code/github/zero-os
cd /opt/code/github/zero-os
git clone https://github.com/zero-os/0-robot.git
cd 0-robot
pip install -e .
```

Check the config manager:
```python
j.tools.configmanager.check()
```

You should see something like:
```
- path: /Users/yves/code/docs.grid.tf/yves/jsconfig
- keyname: id_rsa
- is sandbox: False
- sshagent loaded: True
- key in sshagent: True
````

Copy the ssh/https addresses of the Git repositories in environment variables:
```bash
data_repo="ssh://git@docs.greenitglobe.com:10022/yves/robotdata.git"
dm_templates_repo="https://github.com/jumpscale/digital_me.git"
zos_templates_repo="https://github.com/zero-os/0-templates"
```

> In case you already cloned above template repositories, first make sure you have pull the latests changes.

Start the Zero-Robot:
```bash
zrobot server start --listen :6600 --template-repo $dm_templates_repo --template-repo $zos_templates_repo --data-repo $data_repo
```

> In the above we didn't specify a config repo, in which case the config repo you already configured will be used

Check the configured Zero-Robots:
```bash
zrobot robot list
```

In order to add the one you just started:
```bash
zrobot robot connect local_robot http://localhost:6600
```

Check which Zero-Robot is current:
```bash
zrobot robot current
```

List all services:
```bash
zrobot service list
```

In the JumpScale interactive shell:
```python
j.clients.zrobot.list()
dm_robot_name = 'local_dm_robot'
dm_robot = j.clients.zrobot.robots[dm_robot_name]
```

In order to check or update the attributes of the Zero-Robot configuration instance:
```python
dm_robot_client = j.clients.zrobot.get(dm_robot_name)
dm_robot_client.config
dm_robot_client.config.data_set(key='url', val='http://localhost:6600')
```

Or in order to create another/new configuration instance and client for this or another Zero-Robot:
```python
dm_robot_name = 'local_dm_robot'
dm_robot_cfg = dict(url='http://localhost:6600')
dm_robot_client = j.clients.zrobot.get(instance=dm_robot_name, data=dm_robot_cfg)
```


<a id='#node-robot'></a>

## Access the Node-Robot of your ThreeFold node (optional)

Go to https://capacity.threefoldtoken.com in order to get the IP address and port on which your Node Robot is listening.

If this is the first time:
```python
node_robot_name = 'my node robot'
node_robot_cfg = dict(url='http://10.102.157.1:6600')
node_robot_client = j.clients.zrobot.get(instance=node_robot_name, data=node_robot_cfg)
node_robot = j.clients.zrobot.robots[node_robot_name]
```

Else:
```python
j.clients.zrobot.list()
node_robot_name = 'my node robot'
node_robot_client = j.clients.zrobot.get(instance=node_robot_name)
```

<a id='stream'></a>

## Stream the Node-Robot output of your ThreeFold node (optional)

> Requires that your node is booted with the development flag

First we need to create a Zero-OS configuration instance for your node.

Name of the Zero-OS configuration instance:
```python
zos_instance_name = 'my node'
```

If you already have config instance for your node, get the client and check the configuration:
```python
zos_client = j.clients.zos.get(instance=zos_instance_name, interactive=False)
zos_client.config
```

Optionally, you might want to delete the existing instance with the same name:
```python
j.clients.zos.delete(instance=zos_instance_name)
```

If not, create a new config instance and client:
```python
tf_node_address = '10.102.157.1'
tf_farmer_id = '...'
zos_cfg = {"host": tf_node_address, "port": 6379, "password_": tf_farmer_id}
zos_client = j.clients.zos.get(instance=zos_instance_name, data=zos_cfg)
```

Get the SAL interface of your ThreeFold node and list the containers, you should see the `zrobot` container:
```python
zos_sal = j.clients.zos.sal.get_node(instance=zos_instance_name)
zos_sal.containers.list()
```

To check the flist version that was used:
```python
zos_sal.client.info.version()
```

And finally in another JumpScale interactive shell, stream the Zero-Robot output:
```python
j.clients.zos.list()
zos_instance_name = 'my node'
#zos_client = j.clients.zos.get(instance=zos_instance_name)
zos_sal = j.clients.zos.sal.get_node(instance=zos_instance_name)
zrobot_container = zos_sal.containers.get(name='zrobot')

subscription = zrobot_container.client.subscribe(job='zrobot')
subscription.stream()
```

<a id='reset-node-robot'></a>

## Reset Node Robot (if needed)

```python
zrobot_container = zos_sal.containers.get(name='zrobot')
zrobot_container.client.bash(script='rm -rf /opt/var/data/zrobot/zrobot_data/*').get()
zrobot_container.stop()
zrobot_container.start()
```

You will now have to reactivate the output streams from your node robot.


<a id='delete-vms'></a>

## Delete VMs using the Zero-OS client (if needed)

Normally you should do this through the Digital Me VM service, but in case you don't have the secrets anymore, you'll need to use the Zero-OS client.

```python
zos_client.kvm.list()
uuid1 = 'b506607f-399f-443d-8a67-4ab204079ab4'
uuid2 = 'fe3a4870-e252-441c-accd-2a5939fb9093'

zos_client.kvm.destroy(uuid=uuid1)
zos_client.kvm.destroy(uuid=uuid2)
```

<a id='delete-containers'></a>

## Delete containers using the Zero-OS client (if needed)

```python
j.clients.zos.list()
zos_instance_name = 'my node'
zos_sal = j.clients.zos.sal.get_node(instance=zos_instance_name)
zos_sal.containers.list()
c1 = zos_sal.containers.get(name='gw_23640ca4-aaf8-4859-8a5e-a0f588854a6f')
c2 = zos_sal.containers.get(name='zdb_zdb_local_sdb')
c1.stop()
c2.stop()
```

In the above I had to drop the `gw_` from the name of the GW container:
```python
c1 = zos_sal.containers.get(name='23640ca4-aaf8-4859-8a5e-a0f588854a6f')
c1.stop()
zos_sal.containers.list()
```

Also, after a while the deleted GW container got recreated by the robot...

```python
zrobot_container = zos_sal.containers.get(name='zrobot')
zrobot_container.client.bash(script='rm -rf /opt/var/data/zrobot/zrobot_data/*').get()
zrobot_container.stop()
zrobot_container.start()
```

<a id='hard-reset-node'></a>

## Hard reset your ThreeFold node (if needed)

Wipe the disks:
```bash
dd if=/dev/zero of=/dev/sda bs=1M count=20
```


<a id="zt-client-service"></a>

## Create a ZeroTier client service

Check the service templates available on your Digital Me robot:
```python
dm_robot.templates.uids
```

If you didn't specify (using the `--template-repo $zos_templates_repo` option when starting your local Zero-Robot) the zero-os/0-templates repository (where the ZeroTier client service template is implemented) add the repository:
```python
dm_robot.templates.add_repo(url='https://github.com/zero-os/0-templates') 
```

Alternatively, or in any case it is always a good idea to pull the latest changes from this template repository, using the `checkout_repo` method:
```python
dm_robot.templates.checkout_repo(url='https://github.com/zero-os/0-templates', revision='master')
```

Get the API token of your ZeroTier account:
```python
zt_token =  zt_client.config.data['token_']
```

Check the existing services on your (Digital Me) local Zero-Robot:
```python
dm_robot.services.names
```

Set the name of the ZeroTier client service:
```python
dm_zt_client_service_name = 'myztclient'
```

Optionally delete the existing ZeroTier client service - in case you have the authorization keys (secrets):
```python
dm_robot.services.get(name=dm_zt_client_service_name).delete()
```

Create the ZeroTier client service:
```python
ZOS_ZTCL_UID = 'github.com/zero-os/0-templates/zerotier_client/0.0.1'
zt_data = {'token': zt_token}
dm_zt_client_service = dm_robot.services.create(template_uid=ZOS_ZTCL_UID, service_name=dm_zt_client_service_name, data=zt_data)
```

Next to creating the service on your local Zero-Robot, you will also notice that an additional ZeroTier configuration instance got created with the above name specified:
```python
j.clients.zerotier.list()
```


<a id='create-vm'></a>

## Create virtual machine

Get the node ID of your node from https://capacity.threefoldtoken.com 

https://capacity.threefoldtoken.com/?cru=0&mru=0&hru=0&sru=0&country=Belgium&farmer=yvesfarm

Also the node address can be found there: http://10.102.157.1:6600

Get your public SSH key, in order to authorize it in the VM:
```python
my_sshkey_name = 'id_rsa'
my_sshkey = j.clients.sshkey.get(instance=my_sshkey_name)
```

Create a VM service based on the Digital Me service template for a VM:
```python
node_id = '00e04c680575'
vm_name = 'myfirstvm'
vm_data = {
    'nodeId': node_id,
    'memory': 1024,
    'disks': [{
        'diskType': 'hdd',
        'size': 10,
        'mountPoint': '/mnt',
        'filesystem': 'btrfs',
        'label': 'test',
    }],
    'zerotier': {'id': zt_private_network_id, 'ztClient': dm_zt_client_service_name},
    'image': 'ubuntu',
    'configs': [{'path': '/root/.ssh/authorized_keys', 'content': my_sshkey.pubkey, 'name': 'sshkey'}]
}
dm_vm_service = dm_robot.services.create(template_uid='github.com/jumpscale/digital_me/vm/0.0.1', service_name=vm_name, data=vm_data)

task = dm_vm_service.schedule_action('install')
task.state
task.eco
```

Get the ZeroTier IP address of the VM; give a little time:
```python
r = dm_vm_service.schedule_action(action='info')
vm_zt_ip_addr = r.result['zerotier']['ip']
```

In order to delete the VM:
```python
dm_vm_service.schedule_action(action='delete')
dm_vm_service.delete()
```

Also check the Robot data repo for all services:
```bash
ls /Users/yves/code/docs/yves/robotdata/zrobot_data/github.com/jumpscale/digital_me/vm
```

@TODO ask FR for adding the VMs to the node manager

<a id="create-webapp"></a>

## Create a Web application

Let's first create a SSH connection to your virtual machine:
```python
ssh_name = "my node"
ssh_cfg = dict(addr=vm_zt_ip_addr, port=22, login="root", sshkey=my_sshkey_name, allow_agent=True)
ssh_client = j.clients.ssh.get(instance=ssh_name, data=ssh_cfg, use_paramiko=False)
```

Check the SSH connection:
```python
ssh_client.isconnected
ssh_client.active
ssh_client.execute(cmd="hostname")
```

Execute script over SSH to start web server:
```python
start_webserver = """mkdir -p /mnt/www
cd /mnt/www
echo "Hello world" > index.html
nohup python3 -m http.server &"""

ssh_client.execute(cmd=start_webserver)
```

<a id='public-gw'></a>

## Create a public gateway

https://github.com/Jumpscale/digital_me/tree/master/templates/gateway

```python
DM_GW_UID = 'github.com/jumpscale/digital_me/gateway/0.0.1'

gw_data = {
    'hostname': 'mygw',
    'domain': 'lan',
    'nodeId': node_id,
    'networks': [{
        'name': 'private',
        'type': 'zerotier',
        'public': False,
        'id': zt_private_network_id,
        'ztClient': dm_zt_client_service_name
    }],
}

dm_gw_service = dm_robot.services.find_or_create(template_uid=DM_GW_UID, service_name='dm_gw', data=gw_data)

dm_gw_service.schedule_action('install').wait(die=True)
```

Add proxy:
```python
proxy_name = 'myproxy'
proxy_cfg = {'name': proxy_name, 'host': vm_zt_ip_addr, 'types': ['http'], 'destinations': [{'vm': vm_name, 'port': 8000}]}
dm_gw_service.schedule_action(action='add_http_proxy', args={'proxy': proxy_cfg}).wait(die=True) 
```

Remove it again:
```python
dm_gw_service.schedule_action(action='remove_http_proxy', args={'proxy': proxy_cfg}).wait(die=True) 
```

