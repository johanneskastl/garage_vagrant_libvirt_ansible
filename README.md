# garage_vagrant_libvirt_ansible

This Vagrant setup creates a VM and sets up the
[Garage](https://garagehq.deuxfleurs.fr/) S3-compatible storage solution on it.

Default OS is openSUSE Leap 15.6. Although that can be changed in the
Vagrantfile, please beware that this will break the Ansible provisioning.

## Vagrant

1. You need vagrant obviously. And ansible. And git...
1. Fetch the box, per default this is `opensuse/Leap-15.6.x86_64`, using
   `vagrant box add opensuse/Leap-15.6.x86_64`.
1. Make sure the git submodules are fully working by issuing `git submodule init
   && git submodule update`
1. Run `vagrant up`
1. Run `vagrant ssh` to log in on the VM.
1. Follow the [quickstart
   guide](https://garagehq.deuxfleurs.fr/documentation/quick-start/) by running
   the following commands:

   ```
   $ garage -c /etc/garage/garage.toml status
   ==== HEALTHY NODES ====
   ID                Hostname  Address         Tags  Zone  Capacity          DataAvail
   80f7c2140ebf41c2  garage    127.0.0.1:3901              NO ROLE ASSIGNED
   ```

1. Make sure to replace the ID below with the ID from the command output just
   above (`80f7c2140ebf41c2` in this example).

   ```
   $ garage -c /etc/garage/garage.toml layout assign -z dc1 -c 1G 80f7c2140ebf41c2
   [...]
   $ garage -c /etc/garage/garage.toml layout show
   [...]
   To enact the staged role changes, type:

       garage layout apply --version 1

   You can also revert all proposed changes with: garage layout revert
   $ garage -c /etc/garage/garage.toml layout apply --version 1
   ==== COMPUTATION OF A NEW PARTITION ASSIGNATION ====

   Partitions are replicated 1 times on at least 1 distinct zones.

   Optimal partition size:                     3.9 MB
   Usable capacity / total cluster capacity:   1000.0 MB / 1000.0 MB (100.0 %)
   Effective capacity (replication factor 1):  1000.0 MB

   dc1                 Tags  Partitions        Capacity   Usable capacity
     80f7c2140ebf41c2        256 (256 new)     1000.0 MB  1000.0 MB (100.0%)
     TOTAL                   256 (256 unique)  1000.0 MB  1000.0 MB (100.0%)


   New cluster layout with updated role assignment has been applied in cluster.
   Data will now be moved around between nodes accordingly.
   $ garage -c /etc/garage/garage.toml bucket create vagrant-libvirt-bucket
   $ garage -c /etc/garage/garage.toml bucket list
   List of buckets:
     vagrant-libvirt-bucket    bde5023bf03ba462fcbfd4966fc74e07dc59b1084304259d1b481eab384c1eb3
   $ garage -c /etc/garage/garage.toml key create vagrant-libvirt
   Key name: vagrant-libvirt
   Key ID: GK42738b8b846999f3729bf3f4
   Secret key: 6c68298438cad8e16ce51d2c2dcd83d2a0db2871db878f2e279eb3f9689811e2
   Can create buckets: false

   Key-specific bucket aliases:

   Authorized buckets:
   $ garage -c /etc/garage/garage.toml key list
   $ garage -c /etc/garage/garage.toml key info vagrant-libvirt
   Key name: vagrant-libvirt
   Key ID: GK42738b8b846999f3729bf3f4
   Secret key: (redacted)
   Can create buckets: false

   Key-specific bucket aliases:

   Authorized buckets:
   $ garage -c /etc/garage/garage.toml bucket allow \
     --read \
     --write \
     --owner \
     vagrant-libvirt-bucket \
     --key vagrant-libvirt
   New permissions for GK42738b8b846999f3729bf3f4 on vagrant-libvirt-bucket: read true, write true, owner true.
   $
   ```

1. Add the key ID and the secret key from the `create key` command output to the
   file `ansible/s3cmd_list_bucket.sh`. Run the script from the `ansible`
   directory (given then you have s3cmd installed...).

   ```
   $ ./s3cmd_list_bucket.sh
   =====
   Show bucket contents
   =====
   Upload ansible.cfg
   upload: 'ansible.cfg' -> 's3://vagrant-libvirt-bucket/ansible.cfg'  [1 of 1]
    118 of 118   100% in    0s    40.38 KB/s  done
   =====
   Show bucket contents
   2024-10-24 13:58          118  s3://vagrant-libvirt-bucket/ansible.cfg
   =====
   $
   ```

   The script first shows you the contents of the bucket (which is empty, hence
   no output), then uploads the `ansible.cfg` file and shows you the bucket
   again, this time with content.
1. Party!

## Cleaning up

The VMs can be torn down after playing around using `vagrant destroy`. This will
also remove the file `ansible/s3cmd_list_bucket.sh` as well as the file
`ansible/group_vars/all/garage_random_strings.yml` that was used during the
Ansible deployment.

## License

BSD-3-Clause

## Author Information

I am Johannes Kastl, reachable via git@johannes-kastl.de
