BTS FOG PXE Server Documentation
================================

Backstage has access to a _PC deployment server_. This is useful for deploying copies of Windows/Linux to PCs such as our thin
clients and Office computers. Unlike traditional OS installation from a DVD/USB, the deployment server allows PCs to have their
OSs installed over a network connection; this greatly facilitates OS installation on large numbers of machines.

The principle behind a deployment server is that a "golden image" of a desired OS (Operating System) is installed and customized
on one computer. Using the deployment server, this image is then _captured_ from the PC over the network connection using PXE
(Preboot Execution Environment) booting. PXE is a process whereby the PC boots into a basic temporary OS over the network and uses
this to send/receive a copy of its main OS back to the deployment server. Once the captured image is stored, other computers can
be PXE booted to the server, whereupon this image can be _deployed_ to them, also over the network. This results in the same OS
image being installed on multiple computers, complete with any configuration tweaks. When preparing multiple computers, this
represents a major time saving compared with manually configuring each OS from scratch on each PC.

System components:
------------------

The BTS deployment server has two main components: the server itself, and a client router.

The client router is the connection point for PCs needing to be PXE booted and connects to the University's campus _Docking_
network on one side, and local PC(s) on the other. The router detects when a PC requests a PXE boot and initiates a connection
between it and the deployment server to allow the booting process to take place. This client router takes the form of
the BTS touring Aerohive router (see the _touring_aerohive_router_ repository). The link between this client router and the
server is achieved by the use of an _OpenVPN_ network tunnel. This may be regarded as a "virtual Ethernet cable" running all
the way between the server and the router. For more detailed notes regarding the router's tunnelling connection, see the
_OpenVPN_ section at the end of this document.

The main deployment server (note that this term is interchangeable with _PXE server_) consists of two Ubuntu Linux virtual
machines (VMs) on the _SU Server_ provided by the University's computing services. One of these VMs runs _FOG_
(http://www.fogproject.org), the open-source software used for PXE capture and deployment of OS images over the network.
The other VM takes care of the OpenVPN tunnel connection between the main FOG container and the client route, as described
in more detail in this document's _OpenVPN_ section.

The FOG server features a web-based administration interface. This is accessible from on campus or via the University VPN
using the IP address: `138.38.11.59`. From a device connected to the local network on the client router (BTS Aerohive; see
above), the web interface may be accessed using the IP address `192.168.10.2` Alternatively, the interface is publicly
accessible using the URL: `bts-pxesrv.su.bath.ac.uk`. Credentials are as follows:

Username: [REDACTED]
Password: [REDACTED]

Once logged in, the administration interface gives options to view registered computers ("hosts") and captured OS images
("images"). _Tasks_ can be initiated for specific hosts from this interface. Tasks are used to trigger image capture 
or deployment to/from PCs.

Registering a PC to the deployment server:
------------------------------------------

In order for the deployment server to be able to interface with a given computer for image capture, the computer must be
_registered_ with the server. This is a process whereby the computer's MAC address and system hardware information is loaded
into the FOG software's database.

To register the PC, it should be shut down and connected to the LAN (Local Area Network) connection on the
client Aerohive router. For more information on the Aerohive router's connectivity, see the _touring_aerohive_router_ repository.
Next, the PC should be _PXE booted_. This involves setting the PC's BIOS to use the _onboard NIC_ boot menu option. With this
option, the PC will retrieve a minimal OS over the network from the deployment server. Note that this boot menu option
requires the PC's BIOS to be in _legacy boot mode_. UEFI PXE booting is supported, but tends to be somewhat less predictable
owing to factors such as Secure Boot. UEFI- and Legacy-booted OS images are also not interchangeable.

The minimal PXE OS results in a BTS-themed menu appearing on screen. The menu title clearly indicates whether the PC is
registered to the server. Assuming that the PC is not already registered, use the computer's keyboard to select the
"perform full host registration and inventory" option. Follow the various prompts to enter computer details; these
are stored on the PXE server's FOG database.

When registration is complete, the PC reboots. Logging into the FOG web interface (see above) will now show this computer
under _hosts_. Now that the computer is registered, OS images can be captured from or deployed to it.

Capturing an image from a PC onto the deployment server:
--------------------------------------------------------

Note that this process presupposes that the PC from which the image is to be captured is _registered_ to the deployment
server (see above).

The first stage in preparing the "golden image" is to install Windows (or Linux) as normal onto a known-good PC.
This OS installation is then tweaked as desired ready to be rolled out across other machines: any necessary programs are
installed, colours etc. are customised, and specialist drivers may be installed. Note that, in the case of Windows, the
"fast startup" setting must be disabled, as this prevents the capture process from being executed properly. Once the OS
is installed and customised, the computer should be shut down (_not_ put to "sleep" or "hibernate").

Before the image can be captured from the PC, an image listing must be created in the FOG web interface and assigned to
the PC. Log into the FOG web configuration panel and select the _images_ page. Next, select the option to create a new
image and fill in the OS parameters as appropriate. Lastly, navigate to the _hosts_ page, select the correct PC, and
"assign" the aforementioned image to it. FOG now associates the registered PC with the newly-created image.

To start the image capture process, log into the FOG web interface (see above), select the computer from the "hosts" list,
and navigate to the "tasks" tab. Click the option to capture an image, and the task will be initiated. Next, boot up the
PC, once again selecting the "onboard NIC" boot option in the BIOS. This time, instead of showing a multiple-choice PXE
menu, the image capture process is immediately commenced, resulting in a _Partclone_ progress bar. Partclone is the image
capture software leveraged by FOG (see http://www.partclone.org). Partclone will capture the OS image, store it on the server,
and reboot the PC when completed.

To verify that the image capture completed correctly, log into the server's FOG web interface and load up the _images_ list.
A file size should now have appeared next to the OS image associated with this "golden image" PC, indicating that the
server correctly retrieved a copy of the image.

Deploying an image from the PXE server onto a PC:
-------------------------------------------------

Note that this process may be completed regardless of whether or not the target computer is registered to the deployment
server.

Once the "golden image" has been captured, it may be deployed to other computers. This is done by connecting the target
machine(s) to the local network on the client router (see Aerohive details in _System Components_ section) and
_PXE booting_ it by selecting the _onboard NIC_ boot option in the computer BIOS upon power-up. The PXE boot retrieves
a minimal menu-based "OS" over the network from the deployment server. From this menu, select the "Deploy Image" option
and enter the FOG web interface credentials (see _System components_ section above) when prompted. The image to be deployed
may then be selected from a list. Once an image is selected, the PXE OS uses Partclone (see preceding section) to copy it
across from the server onto the computer's local hard drive / SSD.

When the deployment process has completed, the PC automatically reboots into its new operating system.

Technical details of the PXE booting process:
---------------------------------------------

PXE stands for _Preboot Execution Environment_ and is a method by which a computer can boot into an operating system
_over a network connection_ rather than from a local disk or memory stick. It is noteworthy that this requires
a computer BIOS to suppport this functionality. Thankfully, the vast majority of modern BIOSes support PXE owing to its
widespread use as a tool for deploying operating systems on a large scale in corporate environments.

The particular way in which PXE is employed by the FOG deployment server involves a client PC booting into a minimal
"OS" - Partclone - entirely over a network connection. This minimal OS then handles the transfer of the main operating system image
(Windows/Linux etc.) onto the computer's main internal drive.

In general, booting into a PXE OS involves three stages:

1. The BIOS of the client PC establishes an Ethernet connection using the computer's on-board network card (NIC).
   The BIOS next sends out a _DHCP request_ to the router on the local Ethernet network asking for an IP address.
   DHCP stands for _Dynamic Host Configuration Protocol_ and is the process used by computers for obtaining IP
   addresses automatically from a central router.
   Notably, inside this request is included a couple of DHCP _option flags_ - identified by the numbers 66 and 67 -
   which instruct the router that this DHCP request originates from a computer looking for a PXE server. The
   router responds with an IP address for the client to use, but also includes specific responses for the option
   flags: the IP address of the _PXE server_ and the _boot filename_ which the PC should retrieve from the server
   to use as its boot material.
   
2. Once the client PC has obtained a DHCP IP address from the router, together with details of the PXE server and
   boot filename, the computer's BIOS uses TFTP to fetch the boot file from the PXE server. TFTP
   stands for _Trivial File Transfer Protocol_ and is preferable for low-level network booting applications
   owing to its simplicity.

3. The TFTP-fetched boot file is used to boot into a minimal intermediary "OS" known as a _Network Bootstrap Program_,
   or NBP. The NBP is responsible for creating a menu screen from which various actions may be selected by the user.
   Upon selection of a certain menu entry, the NBP then uses TFTP to fetch a separate operating system kernel
   corresponding to this option. The NBP may therefore be considered to act as a "handover" stage between the
   computer's own BIOS and a standalone operating system kernel.

In the specific instance of the BTS FOG deployment server, the FOG menu presented to client PCs upon a PXE boot
is that of the NBP. Partclone, the software used to clone an operating system onto the computer's physical disk,
represents the separate kernel loaded by the NBP.

Other than Partclone, FOG's NBP may be configured to boot into many different types of "bootable" live images;
this includes "live" Linux distributions which can run a full desktop environment without interfering with the
computer's local disk.

Technical details of the FOG deployment server VM:
--------------------------------------------------

As mentioned in the _System Components_ section of this document, the main deployment server consists of two
virtual machines on the DDaT-provided SU Linux machine in a University server room. To access this machine,
please contact the BTS committee (committee@bts-crew.com). This section focuses on the first virtual machine - that
which runs the main deployment server software. For details on the second virtual machine - that which mediates
the network connectivity between the client PCs and the server VM - see the _OpenVPN_ section below.

The main deployment server VM is implemented as an _LXC Container_ on the SU Server (su-srv-01.bath.ac.uk). This
container runs Ubuntu Linux on which is installed the FOG software stack. For details of how the FOG software is
installed, see the FOG documentation (http://wiki.fogproject.org).

The easiest way to access the main FOG server VM is by logging into the underlying SU Server over SSH and
executing the following command to enter a shell within the VM's LXC container:

`sudo lxc exec bts-pxesrv-03 bash`

Where `sudo` denotes administrative privileges, `lxc` refers to the container software, `bts-pxesrv-03` is the
hostname of the virtual machine, and `bash` refers to the shell which should be opened within the VM.
   
OpenVPN router-server connection details:
-----------------------------------------

The link between the client router (used as the connection point for PXE-booting client PCs; see the _System
components_ section above) and the main FOG deployment server VM is mediated by a secondary container which
acts as a "router" of sorts between the two. This intermediate stage is necessary because of the rather unusual
way in which the deployment system has had to be implemented.

Ordinarily, both the deployment server and the client PCs would effectively be on the same local area network (LAN),
with pretty much unrestricted access between the two. However, in the case of this BTS deployment server, the main
VM resides on a separate server-only network and the client Aerohive router connects to the University's _Docking_
campus-wide network, shared with many other devices. It is therefore necessary to _tunnel_ the connection between
the deployment server and the client router (and ultimately its connected PCs) from the server network to the
Aerohive client router's local area network, effectively bridging the two networks as though a long Ethernet cable
has been run between them.

The topology is as follows:

`Main PXE server VM (bts-pxesrv-03) ->  "Router" VM (bts-pxesrv-02) -> [Wider Uni Network] -> Client Aerohive router -> PC(s) to be imaged`

Since both the main server and routing VMs (`bts-pxesrv-03` and `bts-pxesrv-02`, respectively) reside on the same
physical parent server, the link between these two takes the form of a _bridge interface_. This may be thought of
as a "virtual Ethernet cable" which runs between two VMs. Each VM has an IP address on the "end" of this "Ethernet
cable".

The IP addresses used by each VM on the bridge interface are as follows:

bts-pxesrv-03: `192.168.10.2`
bts-pxesrv-02: '192.168.10.1'

In addition to its above bridge IP address, the router VM, `bts-pxesrv-02`, has an IP address on a separate
interface connecting out to a campus-accessible `138.38.11.xxx` network:

bts-pxesrv-02: `138.38.11.59`

An IPTables _NAT_ rule is in place to allow the inter-VM bridge network to access the wider campus network through
this connection. Functionally, this is what allows the main deployment server VM to access / be accessed by an external
connection from the wired campus Docking network.

The router VM may be accessed in much the same way as the main FOG VM (see above section): The underlying SU Server
machine may be accessed over SSH  (su-srv-01.bath.ac.uk) and the following command can then be run to access a shell
within the VM container:

`sudo lxc exec bts-pxesrv-02 bash`

Where `sudo` denotes administrative privileges, `lxc` refers to the container software, `bts-pxesrv-02` is the
hostname of the virtual machine, and `bash` refers to the shell which should be opened within the VM.

Below is an extract from the result of the `ip a` command when run from within the routing VM. This shows
the interface configuration, as outlined above:
```
16535079: brlocal@if16535080: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:93:88:b1 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.10.1/24 brd 192.168.10.255 scope global brlocal
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe93:88b1/64 scope link 
       valid_lft forever preferred_lft forever
16535083: eth1@if16535084: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:37:76:3d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 138.38.11.59/26 brd 138.38.11.63 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe37:763d/64 scope link 
       valid_lft forever preferred_lft forever
```
As discussed above, the _Aerohive Router_ provides a local connection point for client PCs to PXE boot. This Aerohive router is explained
further in its own documentation, as it is also used to provide routing facilities to small event networks in other Backstage contexts
aside from PXE deployment. This router runs the open-source _OpenWRT_ firmware, allowing it to provide relatively advanced networking
capabilities given its small size and cost. It has a _WAN_ (wide area netwwork) port for connecting to the University's wired Docking
network and a _LAN_ (local area network) for connecting to local devices (PCs / video encoders etc.) requiring external connectivity of
some kind. Crucially, this router _masquerades_ any of its local devices, appearing to the external Docking network as only one single
connection.

The IP address range for devices connected to the LAN side of the Aerohive router is: `10.10.150.xxx`

The router has the address of `10.10.150.254` on its own LAN interface.

Configuration of the Aerohive router can be done by connecting to its web interface at http://10.10.150.254 from a device on the LAN network.
Alternatively, a connection can be made over SSH to configure it via its regular Linux Bash terminal. The latter method is recommended
for OpenVPN configuration (see below).

The final component in the server-client connection topology is the _OpenVPN Tunnel_ between the routing VM and the PC-side
client Aerohive router. This is implemented in the form of an OpenVPN _server_ on the routing VM (`bts-pxesrv-02`) and an OpenVPN
_client_ on the Aerohive client router. These two OpenVPN instances form the encrypted tunnel from the main server VM (`bts-pxesrv-03`)
to the client PC(s). Encryption is achieved by the use of a randomly-generated OpenVPN _pre-shared key_.

Configuration files for both the client and server OpenVPN instances are stored in the `/etc/openvpn` folder on the Aerohive router and
`bts-pxesrv-02` routing VM respectively.

At both ends of the connection, firewall rules are in place to allow traffic from the OpenVPN tunnel to reach the bridge network at the
server side and the Aerohive router LAN network at the client PC side.

Below is an extract from the result of the `sudo lxc list` command when run on the underlying host server. This shows the IP addresses 
present on each VM. The `tun0` IP address on the routing VM (`bts-pxesrv-02`) corresponds to the OpenVPN tunnel over to the client Aerohive
router (see below). In addition, 'brlocal' represents the bridge interface between the two VMs:
```
+----------------------+---------+---------------------------+------+-----------------+-----------+
| bts-pxesrv-02        | RUNNING | 192.168.101.1 (tun0)      |      | CONTAINER       | 0         |
|                      |         | 192.168.10.1 (brlocal)    |      |                 |           |
|                      |         | 138.38.11.59 (eth1)       |      |                 |           |
+----------------------+---------+---------------------------+------+-----------------+-----------+
| bts-pxesrv-03        | RUNNING | 192.168.10.2 (brlocal)    |      | CONTAINER       | 0         |
+----------------------+---------+---------------------------+------+-----------------+-----------+
```
