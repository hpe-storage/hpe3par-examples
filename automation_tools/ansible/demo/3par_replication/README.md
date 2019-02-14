# How to use the HPE 3PAR Ansible module to configure replication on volumes

This is a simple example on how to configure volume replication using the HPE 3PAR Ansible module.

We will the basic steps to create volumes and then setup the Remote Copy group and then add the volumes to the group and then enable replication.

First let's cover some housekeeping.

1. Basic knowledge of Ansible and working with modules.

2. I am assuming that you already have at least two 3PARs within your environment and that the appropriate networking, storage zoning, and remote copy links have been configured between arrays. The HPE 3PAR Ansible module supports both FC and iSCSI environments.

>For more information on configuring Remote Copy, refer to the **HPE 3PAR Remote Copy User Guide**:
>
>https://h20628.www2.hp.com/km-ext/kmcsdirect/emr_na-c03618143-21.pdf

3. You need to be on the latest version of the HPE 3PAR Ansible module.

Verify you have at least **hpe3par-sdk version >1.2.0** and **python-3parclient >4.2.8** or higher.
```
[root@ansible workspace]# pip list | grep 3par
hpe3par-sdk (1.2.0)
python-3parclient (4.2.8)
```

**For new installs:**
```
git clone https://github.com/HewlettPackard/hpe3par_ansible_module/
pip install hpe3par_sdk
```

**For existing installs of the plugin:**

To update to the latest version of the HPE 3PAR Ansible module:
```
git pull https://github.com/HewlettPackard/hpe3par_ansible_module/
pip install –U hpe3par_sdk
```

### Getting Started

Once we have everything setup, let's get started by cloning the replication demo repo:

```
git clone https://github.com/hpe-storage/hpe3par-examples
```


  * Navigate to the **automation_tools/ansible/demo/3par_replication** folder

  * Configure your array details in **properties/remote_copy_properties.yml**

```
storage_system_ip_source: "192.168.1.150"
storage_system_name_source: "local-3par"
storage_system_username_source: "3paradm"
storage_system_password_source: "3pardata"
storage_system_ip_target: "192.168.2.150"
storage_system_name_target: "remote-3par"
storage_system_username_target: "3paradm"
storage_system_password_target: "3pardata"
```

  * Use Ansible vault to encrypt the properties file. **Gotta keep those passwords safe!**

```
[root@ansible 3par_replication]# ansible-vault encrypt properties/remote_copy_properties.yml
New Vault password:
Confirm New Vault password:

```


  * Modify the **remote_copy_demo.yml** to match your environment.

    * **volume size**
    * **CPG**
    * **volume name(s)**
    * **remote copy group name**
    * **target mode**

>Check the remote copy reference guide for detailed information on available parameters.
>
>https://github.com/HewlettPackard/hpe3par_ansible_module/blob/master/Modules/readme.md#hpe3par_remote_copy---manage-hpe-3par-remote-copy

  * Run the playbook:

```
ansible-playbook remote_copy_demo.yml --ask-vault-pass
```

You should see something like this:

```yaml
[root@virt-ansible 3par_replication]# ansible-playbook remote_copy_demo.yml --ask-vault-pass
Vault password:

PLAY [localhost] ********************************************************************************

TASK [Gathering Facts] **************************************************************************
ok: [localhost]

TASK [Load Storage System Vars] *****************************************************************
ok: [localhost]

TASK [Create volume on source] ******************************************************************
changed: [localhost] => (item=demo_volume_1)
changed: [localhost] => (item=demo_volume_2)

TASK [Create Remote Copy Group demo_rcg] ********************************************************
changed: [localhost]

TASK [Add volume to remote copy group] **********************************************************
changed: [localhost] => (item=demo_volume_1)
changed: [localhost] => (item=demo_volume_2)

TASK [remote copy group status] *****************************************************************
ok: [localhost]

TASK [debug] ************************************************************************************
ok: [localhost] => {
    "msg": false
}

TASK [Start remote copy] ************************************************************************
changed: [localhost]

PLAY RECAP **************************************************************************************
localhost                  : ok=8    changed=4    unreachable=0    failed=0
```  

Success!
