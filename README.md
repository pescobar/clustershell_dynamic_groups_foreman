# Clustershell external groups sources from foreman

Define [external group sources](https://clustershell.readthedocs.io/en/latest/config.html#external-group-sources) for [clustershell](http://cea-hpc.github.io/clustershell/) from [Foreman](https://theforeman.org/) or [katello](https://theforeman.org/plugins/katello/)

This is a modification of the ansible foreman inventory in 
[https://github.com/ansible/ansible/blob/devel/contrib/inventory/foreman.py](https://github.com/ansible/ansible/blob/devel/contrib/inventory/foreman.py)

Only the methods `_print_data()` and `parse_cli_args()` in class `ForemanInventory` from the original code are overriden.
Check class `ClusterShellInventory` in [clustershell_dynamic_groups_foreman.py](clustershell_dynamic_groups_foreman.py) to see the changes.

## Installation

Assuming that you have already installed clustershell and you have a config folder `/etc/clustershell` clone this repo inside:

```
git clone https://github.com/pescobar/clustershell_dynamic_groups_foreman.git /etc/clustershell/clustershell_dynamic_groups_foreman
```

Now edit `/etc/clustershell/clustershell_dynamic_groups_foreman/foreman.ini` with your proper foreman url, user and password. e.g.

```
[foreman]
url = https://my-katello-server.com
user = myforemanuser
password = myforemanpassword
ssl_verify = False
```

Define the cache folder in `/etc/clustershell/clustershell_dynamic_groups_foreman/foreman.ini` and create it. e.g.

```
[cache]
path = ~/.cache/clustershell_dynamic_groups_foreman
max_age = 86400

```

`mkdir -p ~/.cache/clustershell_dynamic_groups_foreman`

The cache is valid for 24h (I don't add new foreman hosts too often). Adapt it to your needs if desired.
You can always use the option `--refresh-cache` to force a cache refresh.

### Installing the dependencies using conda

Install the dependencies defined in [requirements.txt](requirements.txt) using conda (you can install the deps systems wide using pip if you prefer and use the system python binary)

```
$> wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh
$> bash Miniconda2-latest-Linux-x86_64.sh -p /opt/miniconda2
$> export PATH=/opt/miniconda2/bin:$PATH
$> pip install -r /etc/clustershell/clustershell_dynamic_groups_foreman/requirements.txt
```

### Check you can contact foreman

```
/opt/miniconda2/bin/python /etc/clustershell/clustershell_dynamic_groups_foreman/clustershell_dynamic_groups_foreman.py -h
/opt/miniconda2/bin/python /etc/clustershell/clustershell_dynamic_groups_foreman/clustershell_dynamic_groups_foreman.py --list-groups
/opt/miniconda2/bin/python /etc/clustershell/clustershell_dynamic_groups_foreman/clustershell_dynamic_groups_foreman.py --all-hosts
```

### Update the clustershell config to fetch external groups from foreman

Edit `/etc/clustershell/groups.conf`

```
[Main]

# Default group source
#default: local
default: foreman

confdir: /etc/clustershell/groups.conf.d $CFGDIR/groups.conf.d

autodir: /etc/clustershell/groups.d $CFGDIR/groups.d

[local]
map: [ -f $CFGDIR/groups ] && f=$CFGDIR/groups || f=$CFGDIR/groups.d/local.cfg; sed -n 's/^$GROUP:\(.*\)/\1/p' $f
all: [ -f $CFGDIR/groups ] && f=$CFGDIR/groups || f=$CFGDIR/groups.d/local.cfg; sed -n 's/^all:\(.*\)/\1/p' $f
list: [ -f $CFGDIR/groups ] && f=$CFGDIR/groups || f=$CFGDIR/groups.d/local.cfg; sed -n 's/^\([0-9A-Za-z_-]*\):.*/\1/p' $f

[foreman]
map: /opt/miniconda2/bin/python /etc/clustershell/clustershell_dynamic_groups_foreman/clustershell_dynamic_groups_foreman.py --group $GROUP
all: /opt/miniconda2/bin/python /etc/clustershell/clustershell_dynamic_groups_foreman/clustershell_dynamic_groups_foreman.py --all-hosts
list: /opt/miniconda2/bin/python /etc/clustershell/clustershell_dynamic_groups_foreman/clustershell_dynamic_groups_foreman.py --list-groups
reverse: /opt/miniconda2/bin/python /etc/clustershell/clustershell_dynamic_groups_foreman/clustershell_dynamic_groups_foreman.py --reverse $NODE
```

And check it's working

`$> nodeset -LL`

### Use foreman groups with clush

Try `/opt/miniconda2/bin/python /etc/clustershell/clustershell_dynamic_groups_foreman/clustershell_dynamic_groups_foreman.py --list-groups` or `nodeset -LL` to get a list of all the
available groups in foreman and then try:

```
$> clush -bw @foreman_hostgroup_compute_nodes 'uname -r'
```
