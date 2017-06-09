Quarry
======

The quarry role allows you to direct storage provisioning operations to a
diverse group of providers using a uniform playbook interface.  For example,
you could configure two storage targets where one is a ceph cluster and the
other is a NetApp Data ONTap cluster.  You could then provision volumes on both
targets starting from a common playbook and changing only the desired target
and the create parameters.  This is useful abstraction that can help integrate
provisioning support into applications or to simplify administration of
heterogeneous infrastructure.

In order to use this role effectively you must first understand its concepts of
operations, backends, and targets.  Operations represent the work you want to
perform.  Backends describe the kinds of storage that can be controlled.
Finally, targets represent specific instances of a backend.  Examples of
operations are: create_volume and delete_volume.  Examples of backends are:
ceph, NetApp Data ONTap and EMC XtremIO.  Examples of targets are: My personal
ceph cluster called TheVault and The NetApp cluster in the Pittsburgh office.

Quarry comes with support for the following operations:
- **create_volume:** Create a new volume
- **delete_volume:** Delete a volume
- **create_snapshot:** Take a snapshot from a volume
- **delete_snapshot:** Delete a snapshot
- **initialize_connection:** Attach a volume to a host
- **terminate_connection:** Detach a volume from a host

The following backends are supported (more are planned):
- **rbd:** Ceph block storage
- **netapp:** NetApp DataONTap
- **xtremio:** EMC XtremIO


Requirements
------------

- Tested with Ansible >= 2.3.0 but may also work with older versions.
- Depends on the aglitke.prism role


Configuration
-------------

To configure the targets, create a directory to store the target definition
files.  The default location is `/etc/prism/targets`.  For each target, create
a configuration file `<target>.yml`.  See the `doc/` directory in this role for
annotated example target configuration files for each supported backend.


Generating a playbook
---------------------
The following playbook can be used to generate a playbook that will create a
volume using a configured 'TheVault' target:
```
# Generate the playbook using the local machine
- hosts: localhost
  roles:
    - aglitke.quarry

  vars:
    # Optional: override if not using the default
    target_definitions_dir: /etc/prism/targets

    # Set the name of the output playbook.  If this variable is omitted, no
    # playbook will be generated.
    generated_playbook: /tmp/quarry-playbook.yml

    # Choose a target for this operation
    target: TheVault

    # Select the operation
    operation: create_volume
    params:
      id: 8dd83a83-5b64-40b6-a2e7-0eb5201fdcb8
      size: 1

```

This generates a playbook in `/tmp/quarry-playbook.yml` which can be executed
with `ansible-playbook` to perform the action:
```
- hosts: thevault.example.com
  remote_user: ceph_admin

  roles:
  - quarry

  tasks:
  - name: Create a volume
    quarry_volume:
      backend: rbd
      config: {u'log': u'/tmp/quarry.log', u'rbd_cluster_name': u'TheVault', u'rbd_secret_uuid': u'a735a6a3-a8bf-4e85-a615-a67bcab23785', u'rbd_ceph_conf': u'/etc/ceph/ceph.conf', u'rbd_pool': u'volumes', u'rbd_user': u'admin'}
      state: present
      id: 8dd83a83-5b64-40b6-a2e7-0eb5201fdcb8
      size: 1
```

To switch to a different target you only need to change one line and the
generated playbook would be configured to use the target's configuration and
the correct ansible modules to create a volume on the new target's backend.
```
target: pittsburghNetapp
```


License
-------

GPLv3


Author Information
------------------

Written by Adam Litke - alitke@redhat.com
- https://github.com/aglitke
