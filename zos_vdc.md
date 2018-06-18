# Setup a Zero-OS Virtual Datacenter

Based on: https://gist.github.com/zaibon/11cd326047a7130777d12abe6376212f

Also see [](17-zos_primitives.md) in order to setup a Zero-OS on OVC first.


Setup:
![](https://docs.google.com/drawings/d/e/2PACX-1vTUfHpHQnpok_O1-OB6WorGyfH8Q2BqJwQ9imOuNdAQk9whwb9LvOrChxNCwp387yj9OQhSx_VbwSIe/pub?w=960&amp;h=720)

Steps:

- [Dependencies](#dependencies)
- [Create the ZeroTier private network](#create-zt-priv-network)
- [Create the ZeroTier admin network](#create-zt-admin-network)
- [Join the ZeroTier admin-network](#join-zt-admin-network)
- [Join the public ZeroTier network](#join-public-zt-network)
- [Create a VDC on OpenvCloud](#create-vdc)
- [Boot a Zero-OS admin node on OpenvCloud](#boot-zos)
- [Authorize ZeroTier join request from your OVC node](#authorize-join)
- [Boot a Zero-OS node on a physical node](#boot-zos2)
- [Install the Zero-Robot Client](#zrobot-client)
- [Create Zero-Robot clients](#zrobot-clients)
- [Stream the Zero-Robot output of the OVC node](#stream)
- [Create a ZeroTier client service on both nodes](#zt-client-services)
- [Create a virtual datacenter](#zos-vdc)
- [Create a virtual disk](#create-vdisk)
- [Create a virtual virtual machine](#create-vm)
- [Create a virtual Web app](#create-webapp)
- [Create port forward for the Web application](#pf-webapp)
- [Create HTTP reverse proxy](#reverse-proxy)

<a id='dependencies'></a>

## Dependencies

```bash
#pip3 install --force-reinstall python-jose==1.3.2
pip3 install --upgrade python-jose
pip3 install --force-reinstall pyghmi==1.0.44
```

<a id="create-zt-priv-network"></a>

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

Set the name of the ZeroTier network you want to use to connect to the public gateway:
Create the network:
```python
zt_private_network_name = 'my_private_network'
```

If this ZeroTier network was already created before, get it using the network id:
```python
zt_private_network_id = '9f77fc393eb910bc'
zt_private_network = zt_client.network_get(network_id=zt_admin_network_id)
```

If not created yet the network, create it:
```python
auto_assign_range = '10.147.20.0/24'
zt_private_network = zt_client.network_create(public=False, name=zt_private_network_name, auto_assign=True, subnet=auto_assign_range)
zt_private_network_id = zt_private_network.id
```

Check the result:
```python
#zt_private_network = zt_client.network_get(network_id=zt_private_network_id)
zt_private_network.config
```

<a id="create-zt-admin-network"></a>

## Create the ZeroTier admin network

Set the name of the ZeroTier network you want to use as your management network:
Create the network:
```python
zt_admin_network_name = 'admin_network'
```

If this ZeroTier network was already created before, get it using the network id:
```python
zt_admin_network_id = '9f77fc393e7fd8b4'
zt_admin_network = zt_client.network_get(network_id=zt_admin_network_id)
```

If not created yet the network, create it:
```python
auto_assign_range = '10.147.19.0/24'
zt_admin_network = zt_client.network_create(public=False, name=zt_admin_network_name, auto_assign=True, subnet=auto_assign_range)
zt_admin_network_id = zt_admin_network.id
```

Check the result:
```python
zt_admin_network = zt_client.network_get(network_id=zt_admin_network_id)
zt_admin_network.config
zt_admin_network.config['ipAssignmentPools']
```


<a id="join-zt-admin-network"></a>

## Join the ZeroTier admin network

Join:
```python
j.tools.prefab.local.network.zerotier.network_join(network_id=zt_admin_network_id)
```

Authorize the join request:
```python
zt_machine_addr = j.tools.prefab.local.network.zerotier.get_zerotier_machine_address()

zt_member = zt_admin_network.member_get(address=zt_machine_addr)
zt_member.authorize()
```


<a id="join-public-zt-network"></a>

## Join the public ZeroTier network


Join:
```python
zt_public_gw_network_id = '9f77fc393e094c66'
j.tools.prefab.local.network.zerotier.network_join(network_id=zt_public_gw_network_id)
```

<a id="create-vdc"></a>

## Create a VDC on OpenvCloud

List all your OpenvCloud configuration clients:
```python
j.clients.openvcloud.list()
```

Set the name of the OpenvCloud client to want to use:
```python
#ovc_instance_name = 'be-gen-1'
ovc_instance_name = 'ch-gen-1'
```

In case this client already exists, get it:
```python
ovc_client = j.clients.openvcloud.get(instance=ovc_instance_name)
```

Or create a new one, for which we first we need a JWT to authenticate against OpenvCloud.
```python
app_id = '...'
app_secret = '...'
iyo_cfg = dict(application_id_=app_id, secret_=app_secret)
iyo_client = j.clients.itsyouonline.get(instance='main', data=iyo_cfg)
jwt_ovc = iyo_client.jwt_get(refreshable=True)
```

Now create the client:
```python
#ovc_url = 'be-gen-1.demo.greenitglobe.com'
#ovc_location = ovc_instance_name
ovc_url = 'ch-lug-dc01-001.gig.tech'
ovc_location = 'ch-lug-dc01-001'
ovc_cfg = dict(address=ovc_url, location=ovc_location, jwt_=jwt_ovc)
ovc_client = j.clients.openvcloud.get(instance=ovc_instance_name, data=ovc_cfg)
```

Get to your OpenvCloud account:
```python
ovc_account_name = 'Account_of_Yves'
ovc_account = ovc_client.account_get(name=ovc_account_name, create=False)
```

Create/get an OpenvCloud cloud space (virtual datacenter)
```python
vdc_name = 'my-zero-space'
cloud_space = ovc_account.space_get(name=vdc_name, create=True)
```

Get the public IP address of the cloud space:
```python
cloudspace_public_ip_address = cloud_space.ipaddr_pub
```

<a id="boot-zos"></a>

## Boot a Zero-OS node on OpenvCloud

On this node we will deploy the gateway.

Boot a VM with Zero-OS in the cloud space:
```python
vm_name = 'my-zos-vm'
iyo_organization = 'zos-training-org'
zos_kernel_params = ['organization={}'.format(iyo_organization), 'development']
#zos_branch = 'v1.2.2'
zos_branch = 'development'

ipxe_url = 'ipxe: https://bootstrap.gig.tech/ipxe/{}/{}/'.format(zos_branch, zt_admin_network_id) + '%20'.join(zos_kernel_params)

zos_vm = cloud_space.machine_create(name=vm_name, memsize=8, disksize=10, datadisks=[50], image='IPXE Boot', authorize_ssh=False, userdata=ipxe_url)
```

Alternatively you can also boot a machine on Packet.net; using this option will implicitly create and return a Zero-OS client.


<a id="authorize-join"></a>

## Authorize ZeroTier join request from your admin node

In order to authorize the join request from the Zero-OS node:
```python
zos_member1 = zt_admin_network.member_get(public_ip=cloudspace_public_ip_address)
zos_member1.authorize()
```

<a id="boot-zos2"></a>

## Boot a Zero-OS node on a physical node

First list all your disk devices:
```bash
diskutil list
```

Format the USB device:
```bash
diskutil eraseDisk FAT32 "ZOS" /dev/disk2
```

As a result the formated disk will get mounted (into `/Volumes/ZOS`), check all mount points:
```bash
mount
```

Copy the EFI file:
```bash
mkdir -p /Volumes/ZOS/EFI/BOOT/
wget -O /Volumes/ZOS/EFI/BOOT/BOOTX64.EFI https://bootstrap.gig.tech/uefi/development/9f77fc393e7fd8b4/organization=zos-training-org%20development
```

Unmount:
```bash
diskutil umount /Volumes/ZOS
```

Now remove the USB from your Mac and boot the node.

Authorize the physical node manually.


<a id='zrobot-clients'></a>

## Create Zero-Robot clients

First get a JWT to connect to your Zero-OS nodes:
```python
memberof_scope = 'user:memberof:{}'.format(iyo_organization)
jwt_zos = iyo_client.jwt_get(scope=memberof_scope, refreshable=True)
```

Create Zero-Robot config instances, first for the node robot of the Zero-OS hosted on OpenvCloud:
```python
robot1_name = 'robot1'
robot1_url = 'http://{}:6600'.format(zos_member1.private_ip)
robot1_cfg = dict(url=robot1_url, jwt_=jwt_zos)
j.clients.zrobot.new(instance=robot1_name, data=robot1_cfg)
```

Then for the physical Zero-OS node:
```python
robot2_name = 'robot2'
robot2_url = 'http://10.147.19.140:6600'
robot2_cfg = dict(url=robot2_url, jwt_=jwt_zos)
j.clients.zrobot.new(instance=robot2_name, data=robot2_cfg)
```

And finally, for the physical Zero-OS node that will host the public gateway, connected to the public ZeroTier GW network:
```python
robot3_name = 'public_gw_robot'
robot3_url = 'http://gw1.robot.threefoldtoken.com:6600'
robot3_cfg = dict(url=robot3_url)
j.clients.zrobot.new(instance=robot3_name, data=robot3_cfg)
```

Create Zero-Robot clients:
```python
robot1_client = j.clients.zrobot.get(instance=robot1_name)
robot1 = j.clients.zrobot.robots[robot1_name]

robot2_client = j.clients.zrobot.get(instance=robot2_name)
robot2 = j.clients.zrobot.robots[robot2_name]

robot3_client = j.clients.zrobot.get(instance=robot3_name)
robot3 = j.clients.zrobot.robots[robot3_name]
```

<a id='stream'></a>

## Stream the Zero-Robot output of the OVC node

First we need to create a Zero-OS configuration instance for the newly booted OpenvCloud node.

Name of the Zero-OS configuration instance:
```python
zos_instance_name = vm_name
```

Optionally, you might want to delete the existing instance with the same name:
```python
j.clients.zos.delete(instance=zos_instance_name)
```

If you already have an updated config instance, get the client:
```python
zos_client = j.clients.zos.get(instance=zos_instance_name, interactive=False)
```

If not, create a new config instance and client:
```python
node_address = zos_member1.private_ip
zos_cfg = {"host": node_address, "port": 6379, "password_": jwt}
zos_client = j.clients.zos.get(instance=zos_instance_name, data=zos_cfg)
```

Get the SAL interface and list the containers:
```python
zos_sal = j.clients.zos.sal.get_node(instance=zos_instance_name)
zos_sal.containers.list()
```

To check the flist version that was used:
```python
zos_sal.client.info.version()
```

Stream the Zero-Robot output - in another JumpScale interactive shell:
```python
j.clients.zos.list()
zos_instance_name = '...'
zos_sal = j.clients.zos.sal.get_node(instance=zos_instance_name)
zrobot_container = zos_sal.containers.get(name='zrobot')

subscription = zrobot_container.client.subscribe(job='zrobot')
subscription.stream()
```

Check the version of the Zero-Robot on the node:
```python
zos_sal.containers.get('zrobot').info
```

<a id="zt-client-services"></a>

## Create a ZeroTier client service on both nodes

Get the API token of your ZeroTier account:
```python
zt_token =  zt_client.config.data['token_']
```

Create a ZeroTier client service on the first node:
```python
zt_data = {'token': zt_token}
zt_client_service_name = 'my_zt_client_service'
robot1.services.find_or_create(template_uid='zerotier_client', service_name=zt_client_service_name, data=zt_data)
```

Do the same for the second node:
```python
robot2.services.find_or_create(template_uid='zerotier_client', service_name=zt_client_service_name, data=zt_data)
```

<a id="zos-vdc"></a>

## Create a virtual datacenter (VDC)

Create a service for the gateway:
```python
priv_gw_service_name = 'gw1'
priv_gw_service = robot1.services.create(template_uid='gateway', service_name=priv_gw_service_name, data={
    'hostname': priv_gw_service_name,
    'domain': 'lan',
    'networks': [{
        'name': 'public_zt',
        'type': 'zerotier',
        'id': zt_public_gw_network_id,
        'public': True,
    }, {
        'name': 'private_zt',
        'type': 'zerotier',
        'ztClient': zt_client_service_name,
        'id': zt_private_network_id,
        'public': False,
    }]
})
# install the gateway and wait for it to be deployed
priv_gw_service.schedule_action('install').wait(die=True)
```

This will join the gateway into your private Zero-Tier network. 

<a id="create-vdisk"></a>

## Create a virtual disk

```python
vdisk_service_name = 'vdisk1'
vdisk_service = robot2.services.create(template_uid='vdisk', service_name=vdisk_service_name, data={
        'size': 10,
        'diskType': 'HDD' # can be HDD or SSD
})
vdisk_service.schedule_action('install').wait(die=True)
```

<a id="create-vm"></a>

## Create virtual machine

Get your public SSH key, in order to authorize it in the VM:
```python
my_sshkey = j.clients.sshkey.get(instance='id_rsa')
```

Retrieve the url of the vdisk, needed to mount the vdisk inside the VM:
```python
vdisk_private_url = vdisk_service.schedule_action('private_url').wait(die=True).result
```

Generate a ZeroTier identity for the virtual machine, this will allow later to easily get the ZeroTier IP address:
```python
zm_zt_identity = j.tools.prefab.local.core.run('zerotier-idtool generate')[1].strip()
```

See: https://github.com/zero-os/0-templates/tree/master/templates/vm

Create the virtual machine:
```python

vm_data = {
    'flist': 'https://hub.gig.tech/gig-bootable/ubuntu:16.04.flist',
    'memory': 512,
    'cpu': 1,
    'disks': [{
        'name': vdisk.name,
        'url': vdisk_private_url,
    }],
    'ztIdentity': zm_zt_identity,
    'nics': [{
        'name': 'private_zt',
        'type': 'zerotier',
        'ztClient': zt_client_service_name,
        'id': zt_private_network_id
    }],
    'configs': [{
        'name': 'sshkeys',
        'path': '/root/.ssh/authorized_keys',
        'content': my_sshkey.pubkey
    }],
}
vm_service_name = 'vm1'
vm_service = robot2.services.create(template_uid='vm', service_name=vm_service_name, data=vm_data)
vm_service.schedule_action('install').wait(die=True)
```

<a id="create-webapp"></a>

## Create a Web application

SSH into the VM and execute:
```bash
mkfs.ext4 /dev/vda
mount /dev/vda /mnt

mkdir -p /mnt/www
cd /mnt/www
echo "Hello world" > index.html
python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

<a id="pf-webap"></a>

## Create port forward for the Web application

```python
vm_zt_ip = '10.147.20.232'
priv_gw_service.schedule_action(action='add_portforward', args={'forward':{
            'srcport': 8000,
            'srcnetwork': 'public_zt',
            'dstip': vm_zt_ip,
            'dstport': 8000,
            'name': 'webapp_pf'
}}).wait(die=True)
```

<a id="reverse-proxy"></a>

## Create HTTP reverse proxy

First we need to get the ZT IP address of the private gateway in the public ZeroTier network:
```python
import netaddr
priv_gw_zt_address = None
info = priv_gw_service.schedule_action('info').wait(die=True).result
for network in info['networks']:
    if network['name'] == 'public_zt':
        nw = netaddr.IPNetwork(network['config']['cidr'])
        priv_gw_zt_address = str(nw.ip)
        break
# ensure that you have the gateway ip
assert priv_gw_zt_address is not None
```

Let's create the HTTP reverse proxy now on the public gateway:
```python
rproxy_data = {
    'httpproxies': [{
        'host': 'www.vreegoebezig.be', # replace this with the domain you configured earlier
        'destinations': ['http://{}:8000'.format(priv_gw_zt_address)],
        'types': ['http','https'],
        'name': 'webapp_http_proxy_yves'
    }]
}
public_gw_service_name = 'pubgw1'
public_gw_service = robot3.services.create(template_uid='public_gateway', service_name=public_gw_service_name, data=rproxy_data)
public_gw_service.schedule_action('install').wait(die=True)
```