# dbimage-builder Charm #

The dbimage-builder charm utilizes the dbimage-builder tool to create a
bootable virtual disk image that is configured to provide a user specified 
database.


## Deployment ##

### Deployment Requirements ###

The dbimage-builder charm is a subordinate charm of trove, so the trove charm
will need to be installed before deployment. The charm also requires access 
to openstack for creating a virtual machine. This virtual machine will be used
for creating images and will need an ubuntu image, trove tenant network
and an external network.

During deployment, the charm will attempt to use configuration defaults to
create the VM. If this fails, the charm will monitor configuration updates
and attempt to create the VM again.

Defaults:
  vm_image     = xenial-1604
  external_net = external_net
  trove_net    = trove_net 

### Deployment Commands ###

> git clone https://github.com/open-power-ref-design-toolkit/dbimage-builder-charms

> juju deploy dbimage-builder-charms/dbimage-builder

> juju add-relation dbimage-builder trove


## Configuration ##

vm_image:
Ubuntu image used to create the virtual machine. Defaults to xenial-1604

trove_net:
OpenStack network used by trove tenants. Defaults to trove_net.

external_net:
OpenStack network used to connect externally. Defaults to external_net.


## Actions ##

### dbimage-make ###

> juju run-action dbimage-builder/<unit_number> dbimage-make db=<db-name>
>    [ version=<version> ] [ c=True | e=True ] [ keyname=<keyname> ]

Creates a bootable O/S image containing the named database and database version
and creates an OpenStack Trove datastore from the image. The image is built 
using the OpenStack diskimage-builder (DIB) project.

The **db** argument must be one of mariadb, mongodb, mysql, postgresql, or redis.

The **c** argument may be specified to select the community edition of a database
if one is provided and supported by this tool. The **e** argument may be 
specified to select the enterprise edition of a database if one is provided 
and supported by this tool. When the command arguments c and e are not 
specified, the selection defaults to a distro provided database if one is 
provided and supported by the tool.

The **keyname** argument names a ssh key pair that is registered with OpenStack.
If this argument is specified, then the public ssh key is obtained from 
OpenStack and is placed in virtual disk image in the file 
/home/ubuntu/.ssh/authorized_keys. This is intended for DBA access.

### dbflavor Actions ###

These actions are used after the datastore has been created by the 
dbimage-make action. They are used to create the datastore flavors which
dictate the capacity of datastore instances -- vcpus, ram, and storage.

The following actions are used to show, change, and upload database flavors
for glance images created by the dbimage-make action:

> juju run-action dbimage-builder/<unit_number> dbflavor-show db=<db-name>
>    [ predefined=True ]

> juju run-action dbimage-builder/<unit_number> dbflavor-change db=<db-name>
>    flavor=flavor-name { [ vcpus=<val> ] | [ mem=val ] | [ vdisk=<val> ] }

> juju run-action dbimage-builder/<unit_number> dbflavor-upload db=<db-name>
