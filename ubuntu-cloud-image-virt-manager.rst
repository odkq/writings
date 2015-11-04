Using Ubuntu Cloud Images with KVM and virt-manager
===================================================

Suppose you have to test/build something on Ubuntu 12.04 LTS, you have
a modern distribution and do not want to install a virtual image from the
CD. Ubuntu releases 'cloud images' wich are the same they distribute for
EC2, Azure, etc. They can be used directly with kvm and virt-manager, with
some caveats;

Getting the image and running it
--------------------------------

Download the image from http://cloud-images.ubuntu.com/releases/12.04.2/release/ubuntu-12.04-server-cloudimg-amd64-disk1.img

Copy the image to /var/lib/libvirt/images

Launch virt-manager and create a new image (File/New virtual machine). Select
the image downloaded.

It will take some time to launch the first time, with a blank screen. This
is normal as it is configuring itself. In theory, in the text console a
randomly choosen password should appear as ubuntu_pass = <whatever>. But
it doesn't.

Changing the ubuntu user password
---------------------------------

Following http://ubuntu-smoser.blogspot.fr/2013/02/using-ubuntu-cloud-images-without-cloud.html do::

 cat > my-user-data <<EOF
 #cloud-config
 password: passw0rd
 chpasswd: { expire: False }
 ssh_pwauth: True
 EOF

 cloud-localds my-seed.img my-user-data

Configure the image with my-seed.img as second disk (Add hardware, disc, etc)

Upon launch, the password should change (in this case, ubuntu:passw0rd)

Remove the auxiliar disk, as otherwise the image won't be cloned.

Resizing the image
------------------

The image has 2.2 GB by default, to give it more room, go to
/var/lib/libvirt/images and do::

 qemu-img resize <image> +10G

Configuring the network bridge
------------------------------
This will give the image an address in a bridged lan, so we can connect
to the image through ssh, etc. (Otherwise it uses the kvm internal networking,
that does not expose a WAN and only allows for outgoing connections)

virsh net-start default

On the image configuration, in the NIC, 'Specify shared device name: virbr0'

