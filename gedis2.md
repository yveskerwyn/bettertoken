# Getting started with Gedis2

Also see: https://github.com/rivine/recordchain/tree/master/JumpScale9RecordChain/servers/gedis2

Steps:
- [](#)


Get JumpScale installed:
```bash
ssh -A root@195.134.212.38 -p7522
export ZUTILSBRANCH=development
curl https://raw.githubusercontent.com/Jumpscale/bash/${ZUTILSBRANCH}/install.sh?$RANDOM > /tmp/install.sh;bash /tmp/install.sh
source ~/.bashrc
export JS9BRANCH=development
ZInstall_host_js9_full
```

Also see: https://github.com/Jumpscale/bash#install-jumpscale


Initialize your configuration manager, as documented here:
https://github.com/threefoldfoundation/info_tfgrid_tutorial/blob/master/init_config_manager.py




Dependencies:
```bash
#apt install libssl-dev
#pip3 install python-jose
pip3 install cryptocompare
#pip3 install pyblake2
#pip3 install pycapnp
```

Install the recordchain:
```bash
mkdir -p /opt/code/github/rivine
cd /opt/code/github/rivine
git clone https://github.com/rivine/recordchain.git
cd /opt/code/github/rivine/recordchain
sh install.sh
```

Install 0-db:
```bash
cd /opt/code/github/rivine
git clone https://github.com/rivine/0-db.git
cd /opt/code/github/rivine/0-db
make
cp bin/zdb /opt/bin/
```

Or use Docker:
```bash
docker run --privileged --hostname gedis --name gedis -v /opt/code:/opt/code -i -d -t ubuntu:18.04
```

Then open multiple instances of it using `docker exec -it gedis /bin/bash`

I make sure to install in docker vim, net-tools, sudo


Start a new TMUX session:
```bash
tmux new -s gedis2
```

Make your own server:
```python
app_name = 'my_gedis_app'
#app_dir = '/opt/var/tfgrid'
app_dir = ''
app_cfg = dict(
    adminsecret_ = '',
    apps_dir = app_dir,
    host = 'localhost',
    port = '9971',
    ssl = 'false')

app = j.servers.gedis2.get(instance=app_name, data=app_cfg)
app.start(background=False, reset=True)
```

> Using `app_dir = '/opt/var/tfgrid'` doesn't work...
> When using `app_dir = ''` it will default to `/opt/code/github/rivine/recordchain/JumpScale9RecordChain/apps/`

As a result a new `{apps_dir}/{instance_name}` directory will be created: `/opt/var/tfgrid/my_gedis_app/`. It contains only one empty file `__init__.py`.

Also code gets generated in `/opt/var/`

Split screen (`CTRL+B "`) and again in a new interactive shell, configure and get a client:
```python
app_name = 'my_gedis_app'
app_cfg = dict(
    adminsecret_ = '',
    host = 'localhost',
    port = '9971',
    ssl = 'false')
app_client = j.clients.gedis2.get(instance=app_name, data=app_cfg, reset=True)
app_client.system.ping()
```

In order to check the config:
```python
app_cl._client.config
```

Create a new `mycommands.py` in `/opt/code/github/rivine/recordchain/JumpScale9RecordChain/apps/my_gedis_app`:
```python
from js9 import j

JSBASE = j.application.jsbase_get_class()

class mycommands(JSBASE):
    def __init__(self):
        JSBASE.__init__(self)

    def echo(self, str):
        return str
```

> Make sure the name of the file is the same as the name of the class.

Stop the server, and restart.

from the client:
```python
app_name = 'my_gedis_app'
app_client = j.clients.gedis2.get(instance=app_name)
app_client.mycommands.echo('hello world') )
```

Create a `schema.toml` in `/opt/code/github/rivine/recordchain/JumpScale9RecordChain/apps/my_gedis_app`:
```toml
@url = tfgrid.vm.create
@name = createvm
node_id = "" (S)
vm_name = "" (S)
```

Create `tfgrid.py` in `/opt/code/github/rivine/recordchain/JumpScale9RecordChain/apps/my_gedis_app`:
```python
from js9 import j

JSBASE = j.application.jsbase_get_class()

class tfgrid(JSBASE):
    def __init__(self):
        JSBASE.__init__(self)

    def create_vm(self, vm_data):
        """
        some test method, which returns something easy
        ```in
        !tfgrid.vm.create
        ```
        """
        return True
```

Restart the gedis app.

from the client:
```python
app_name = 'my_gedis_app'
app_client = j.clients.gedis2.get(instance=app_name)
app_client.tfgrid.create_vm?
```

Read about schemas: https://github.com/rivine/recordchain/blob/master/JumpScale9RecordChain/data/schema/README.md

Define a new schema:
```python
schema = """
  @url = vreegoebezig.tfgrid.createvm
  @name = createvm
  nodeid = "" (S)             
  vmname = "" (S)                              
  """ 
```
  
List of schemas returned from 
```python
schemas = j.data.schema.schema_add(schema)
schema = schemas[0]
```