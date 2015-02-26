##**BEFORE YOU ARRIVE**
    In order to make best use of the lab time please review the deployment options and ensure either 

1. A working KVM environment or
2. Access to the internal OpenStack OS1 environment.

##**Agenda / High Level Overview:**

1. Deploy Atomic Hosts
2. Configure Flannel
3. Configure K8s
4. Deploy an Application
5. Use SPCs on the Atomic Hosts


#**Deployment**
There are many ways to deploy an Atomic host.  In this lab, we provide guidance for OpenStack or local KVM.

##**Deployment Option 1: Atomic Hosts on OpenStack**
You may use the the Red Hat internal, HSS-supported **OS1** OpenStack service.

1. PREREQUISITE: You must create an OS1 account and upload your SSH public key. Follow the ["Getting Started" instructions](https://mojo.redhat.com/docs/DOC-28082#jive_content_id_Getting_Started)
1. Navigate to [Instances](https://control.os1.phx2.redhat.com/dashboard/project/instances/)
1. Click "Launch Instances"
1 Complete form
  1. Details tab
    * Instance name: arbitrary name. Note the UUID of the image will be appended to the instance name. You may want to use your name in the image so you can easily find it.
    * Flavor: *m1.medium*
    * Instance count: *3*
    * Instance Boot Source: *Boot from image*
      * Image name: *rhel-atomic-cloud-7.1-6*
  1. Access & Security tab
    * select you keypair that was uploaded during OS1 account setup. See [instructions](https://mojo.redhat.com/docs/DOC-28082#jive_content_id_Getting_Started)
    * Security Groups: *Default*
1. Click "Launch"

Three VMs will be created. Once Power State is *Running* you may SSH into the VMs. Your SSH public key will be used.

* Note: Each instance requires a floating IP address in addition to the private OpenStack `172.x.x.x` address. Your OpenStack tenant may automatically assign a floating IP address. If not, you may need to assign it manually. If no floating IP addresses are available, create them.
  1. Navigate to [Access & Security](https://control.os1.phx2.redhat.com/dashboard/project/access_and_security/)
  1. Click "Floating IPs" tab
  1. Click "Allocate IPs to project"
  1. Assign floating IP addresses to each VM instance
* SSH into the VMs with user `cloud-user` and the instance floating IP address. This address will be in the `10.3.xx.xx` range.

```
ssh cloud-user@10.3.xx.xxx
```


##**Deployment Option 2: Atomic Hosts on KVM**

* Grab and extract the Atomic and metadata images from our internal repo.  Use sudo and appropriate permissions.

```
wget http://refarch.cloud.lab.eng.bos.redhat.com/pub/projects/atomic/atomic0-cidata.iso
cp atomic0-cidata.iso /var/lib/libvirt/images/.
wget http://download.eng.bos.redhat.com/rel-eng/Atomic/7/trees/GA.brew/images/20150217.0/cloud/rhel-atomic-host-7.qcow2.gz
cp rhel-atomic-host-7.qcow2.gz /var/lib/libvirt/images/.; cd /var/lib/libvirt/images
gunzip rhel-atomic-host-7.qcow2.gz
```

* Make 3 copies of the image.

```
cp rhel-atomic-host-7.qcow2 rhel-atomic-host-7-1.qcow2
cp rhel-atomic-host-7.qcow2 rhel-atomic-host-7-2.qcow2
cp rhel-atomic-host-7.qcow2 rhel-atomic-host-7-3.qcow2
```

* Use the following commands to install the images. Note: You will need to change the bridge to match your setup, or at least confirm it matches what you have.

```
virt-install --import --name atomic-ga-1 --ram 4096 --vcpus 2 --disk path=/var/lib/libvirt/images/rhel-atomic-host-7-1.qcow2,format=qcow2,bus=virtio --disk path=/var/lib/libvirt/images/atomic0-cidata.iso,device=cdrom --network bridge=br0 --force

virt-install --import --name atomic-ga-2 --ram 4096 --vcpus 2 --disk path=/var/lib/libvirt/images/rhel-atomic-host-7-2.qcow2,format=qcow2,bus=virtio --disk path=/var/lib/libvirt/images/atomic0-cidata.iso,device=cdrom --network bridge=br0 --force

virt-install --import --name atomic-ga-3 --ram 4096 --vcpus 2 --disk path=/var/lib/libvirt/images/rhel-atomic-host-7-3.qcow2,format=qcow2,bus=virtio --disk path=/var/lib/libvirt/images/atomic0-cidata.iso,device=cdrom --network bridge=br0 --force
```

##**Update VMs**

**NOTE:** We will be working on _all three (3)_ VMs. You will probably want to have three terminal windows open.

* Confirm you can login to the hosts:

    Username: cloud-user
    Password: atomic (KVM only)

* Enter sudo shell:

```
sudo -i
```


* Update all of the atomic hosts. The following commands will change you to the GA.staging tree, which should be what customers will see.

```
atomic host status
ostree remote add --set=gpg-verify=false GA.brew http://download.eng.bos.redhat.com/rel-eng/Atomic/7/trees/GA.brew/repo/
rpm-ostree rebase GA.brew:rhel-atomic-host/7/x86_64/standard
```
This will fetch a new tree and display any RPM changes. **Note:** Customers will not have to add a remote. They will issue command `atomic host upgrade` then reboot.

* Check the atomic tree version

```
atomic host status
```

Note the `*` identifies the active version.

* Reboot the VMs to switch to updated tree.

```
systemctl reboot
```

* After the VMs have rebooted, SSH into each and enter sudo shell:

```
sudo -i
```

* Check your version with atomic. The `*` pointer should now be on the new tree.

```
atomic host status
```
