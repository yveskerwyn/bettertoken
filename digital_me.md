# Digital Me

- [Bootstrap](#bootstrap)

- [Create the VM machine](#create-vm)

<a id='bootstrap'></a>

## Bootstrap

See: https://github.com/Jumpscale/digital_me/tree/master/specs

Digital Me is actually a set of service templates for getting things done in the ThreeFold Grid.

In order to use these Digital Me service templates you need a Zero-Robot, which can be seen as your Digital Me.

So first step is to get a running instance of a Zero-Robot on your local machine, or any (virtual) machine you have admin access too.

For this follow the steps as documented in https://github.com/yveskerwyn/jumpscale/blob/master/12-zero-robot.md

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
robot_name = 'local_robot'
robot = j.clients.zrobot.robots[robot_name]
```

<a id="zt-client-services"></a>

## Create a ZeroTier client service

First (if not used `--template-repo $zos_templates_repo` when starting the robot) add the zero-os/0-templates repository, since this where the ZeroTier client service template is implemented:
```python
robot.templates.add_repo(url='https://github.com/zero-os/0-templates') 
```

Alternatively, in case you just need to pull the latest changes from this template repository use the `checkout_repo` method:
```python
robot.templates.checkout_repo(url='https://github.com/zero-os/0-templates', revision='master')
```

Get the API token of your ZeroTier account:
```python
zt_token =  zt_client.config.data['token_']
```

Set the name of the ZeroTier client service:
```python
zt_client_service_name = 'myztclient'
```

Optionally delete the existing ZeroTier client service - in case you have the authorization keys (secrets):
```python
robot.services.get(name=zt_client_service_name).delete()
```

Create the ZeroTier client service:
```python
zt_data = {'token': zt_token}
zt_client_service = robot.services.create(template_uid='github.com/zero-os/0-templates/zerotier_client/0.0.1', service_name=zt_client_service_name, data=zt_data)
```


<a id='create-vm'></a>

## Create virtual machine

Get the node ID of your node from https://capacity.threefoldtoken.com 

https://capacity.threefoldtoken.com/?cru=0&mru=0&hru=0&sru=0&country=Belgium&farmer=yvesfarm

Also the node address can be found there: http://10.102.157.1:6600

Get your public SSH key, in order to authorize it in the VM:
```python
my_sshkey = j.clients.sshkey.get(instance='id_rsa')
```


```python
node_id = '00e04c680575'
zt_private_network_id = 'e4da7455b2dc3436'
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
    'zerotier': {'id': zt_private_network_id, 'ztClient': zt_client_service_name},
    'image': 'ubuntu',
    'configs': [{'path': '/root/.ssh/authorized_keys', 'content': my_sshkey.pubkey, 'name': 'sshkey'}]
}
vm_service = robot.services.create(template_uid='github.com/jumpscale/digital_me/vm/0.0.1', service_name=vm_name, data=vm_data)

task = vm_service.schedule_action('install')
task.state
task.eco
```

Get the IP address of the VM:
```python
r = vm_service.schedule_action(action='info')
r.result
```

In order to delete the VM:
```python
vm_service.schedule_action(action='delete')
vm_service.delete()
```

Also check the Robot data repo for all services:
```bash
ls /Users/yves/code/docs/yves/robotdata/zrobot_data/github.com/jumpscale/digital_me/vm
```

@TODO ask FR for adding the VMs to the node manager

<a id='stream'></a>

