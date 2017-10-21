[[!meta title="KVM/Libvirt"]]

Most of the OCF's hosts are virtual machines. Our virtual machines are all
powered by [QEMU][qemu]/[KVM][kvm] and managed by [libvirt][libvirt] on the
hypervisors.

VM disks are stored as LVM volumes on the hypervisors, typically under
`/dev/vg/<vm-name>`.

[qemu]: https://www.qemu.org/
[kvm]: https://www.linux-kvm.org/
[libvirt]: https://libvirt.org/


## Administration

### How do I view which VMs are on a hypervisor?

SSH into the hypervisor, then run `sudo virsh list --all`:

    kpengboy@hal:~$ sudo virsh list --all
     Id    Name                           State
    ----------------------------------------------------
     1     fallingrocks                   running
     3     limniceruption                 running
     4     spriggs                        running
     5     dev-flood                      running
     11    dev-whiteout                   running
     15    dev-death                      running
     18    dev-tsunami                    running
     -     dev-anthrax                    shut off
     -     dev-pestilence                 shut off
     -     dev-werewolves                 shut off
     -     zombies                        shut off

### How do I turn on a VM?

On the hypervisor, run `sudo virsh start <vm-name>`.

### How do I turn off a VM?

You can SSH into the VM and run the `shutdown` command, or you can run
`sudo virsh stop <vm-name>` on the hypervisor which hosts it.

If it's a public-facing VM (e.g. tsunami), remember to give a positive amount
of time to the shutdown command, so users have adequate warning.

### How do I make a VM automatically turn on when the hypervisor boots?

On the hypervisor, run `sudo virsh autostart <vm-name>`.

You can list which VMs are set to autostart with `sudo virsh list --all
--autostart`.

Firestorm is set to autostart because it must be running in order for any staff
to log in (other than by using the root account). Other VMs are not set to
autostart because if they start before LDAP and Kerberos are available, logins
may not necessarily work properly.

### Is there a GUI for all of this?

You can run virt-manager on the hypervisors to graphically start and stop VMs,
view their virtual monitors, and do many other similar things. To access
virt-manager, SSH into the hypervisor with X forwarding (`-X`) enabled. Then, on
the hypervisor, run `sudo XAUTHORITY=~/.Xauthority virt-manager`.

#### If you just want to see a single VM's virtual monitor

You can use the virt-viewer tool. Replace `virt-manager` in the above command
with `virt-viewer <vm-name>`.

Alternatively, you can directly connect to the VM's VNC display. This procedure
is more complicated, but gives much better performance than X forwarding.
TODO (jvperrin)

### How do I create a VM?

See [[Creating new hosts|doc staff/procedures/new-host]].

### How do I delete a VM?

On the hypervisor:

1. Shutdown the VM.
2. Run `sudo virsh undefine <vm-name>`.
3. Backup the VM's disk (e.g. by renaming the LVM volume to `vg/<vm-name>.old`)
   or delete it. You may want to also dump the contents of the disk to a file,
   compressing it, and placing that file in `/opt/backups/live/misc/servers` on
   the server which contains backups (which is `hal` at the time of this
   writing). You may also want to save the VM's XML definition by running
   `sudo virsh dumpxml [vm-name] > [vm-name].xml` and placing it in the same
   aforementioned directory.

### How do I move a VM from one host to another?

Use the [[migrate-vm|doc staff/scripts/migrate-vm]] script.

### Oh no, I've got a VM with broken networking. How can I access it?

You can open virt-manager as described above, open the VM's display, and then
log in there.

Alternatively, if you don't want to bother with opening up a GUI, you can often
access the VM using its serial console. Run `sudo virsh console <vm-name>` on
the hypervisor. This will connect you to the VM's TTY listening on its simulated
serial port.

You may have to initially hit enter for the VM to (re-)print the login prompt.
When you're done, make sure to log out. Then use Ctrl+] to exit the virsh
console.

This method of accessing the VM only works when there is a getty process
listening on the serial port. All of our VMs start such a process automatically,
but only after boot has mostly finished. Therefore, the serial console probably
won't work if you're trying to diagnose boot problems.

### How do I edit my VM's RAM size or CPU count?

On the hypervisor, run `sudo virsh edit <vm-name>` to edit the VM's XML
definition.

### How do I edit my VM's disk size?

TODO