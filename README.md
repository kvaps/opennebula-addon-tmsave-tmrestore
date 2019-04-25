# OpenNebula tmsave / tmrestore addon

In contionue to this discussion https://forum.opennebula.org/t/shared-mode-for-the-ceph-based-system-datastires/7164/9

This addon to adds `tm/presave` `tm/postsave` `tm/prerestore` `tm/postrestore` actions call to vmm driver.


So saving VM state will looks like:

|#|driver|action|execution|description|
|-|-|-|-|-|
|1|tm|presave|frontend|preparing the place and setups checkpoint file symlink|
|2|vmm|save|remote|saves VM state  into checkpoint file (or device) under symlink|
|3|tm|postsave|frontend|optional disconnects the device with checkpoint from the compute-node|

Restoring:

|#|driver|action|execution|description|
|-|-|-|-|-|
|1|tm|prerestore|frontend|connects the device with checkpoint to the compute-node, and setups symlink on it|
|2|vmm|restore|remote|restores VM action from the symlink|
|3|tm|postrestore|frontend|removes old checkpoint file|


## Installation

1. put [tmsave](tmsave) and [tmrestore](tmrestore) into to your vmm driver path, eg. `/var/lib/one/remotes/vmm/kvm`
2. Update your VM_MAD in oned.conf

   ```diff
    VM_MAD = [
        NAME           = "kvm",
        SUNSTONE_NAME  = "KVM",
        EXECUTABLE     = "one_vmm_exec",
   -    ARGUMENTS      = "-t 15 -r 0 kvm",
   +    ARGUMENTS      = "-t 15 -r 0 kvm -l save=tmsave,restore=tmrestore",
        DEFAULT        = "vmm_exec/vmm_exec_kvm.conf",
        TYPE           = "kvm",
        KEEP_SNAPSHOTS = "yes",
        IMPORTED_VMS_ACTIONS = "terminate, terminate-hard, hold, release, suspend,
            resume, delete, reboot, reboot-hard, resched, unresched, disk-attach,
            disk-detach, nic-attach, nic-detach, snapshot-create, snapshot-delete"
    ]
   ```

3. Restart OpenNebula service
