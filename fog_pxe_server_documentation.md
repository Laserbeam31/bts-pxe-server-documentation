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
end of this document.

The main deployment server (note that this term is interchangeable with _PXE server_) consists of two Ubuntu Linux virtual
machines (VMs) on the _SU Server_ provided by the University's computing services. One of these VMs runs _FOG_
(http://www.fogproject.org), the open-source software used for PXE capture and deployment of OS images over the network.
The other VM takes care of the OpenVPN tunnel connection between the main FOG container and the client router. For more
information about these virtual machines, see the end of this document.

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

In order for the deployment server to be able to interface with a given computer, either for image capture or deployment,
the computer must be _registered_ with the server. This is a process whereby the computer's MAC address and system
hardware information is loaded into the FOG software's database.

To register the PC, it should be shut down and connected to the LAN (Local Area Network) connection on the
client Aerohive router. For more information on the Aerohive router's connectivity, see the _touring_aerohive_router_ repository.
Next, the PC should be _PXE booted_. This involves setting the PC's BIOS to use the "onboard NIC" boot menu option. With this
option, the PC will retrieve a minimal OS over the network from the deployment server. Note that this boot menu option
requires the PC's BIOS to be in _legacy boot mode_. UEFI PXE booting is supported, but tends to be somewhat less predictable
owing to factors such as Secure Boot. UEFI and Legacy OS images are also not interchangeable.

The minimal PXE OS results in a BTS-themed menu appearing on screen. The menu title clearly indicates whether the PC is
registered to the server. Assuming that the PC is not already registered, use the computer's keyboard to select the
"perform full host registration and inventory" option. Follow the various prompts to enter computer details; these
are stored on the PXE server's FOG database.

When registration is complete, the PC reboots. Logging into the FOG web interface (see above) will now show this computer
under "hosts". Now that the computer is registered, OS images can be captured from or deployed to it.

Capturing an image from a PC onto the deployment server:
--------------------------------------------------------

Note that this process presuppposes that the PC from which the image is to be captured is _registered_ to the deployment
server (see above).

The first stage in preparing the "golden image" is to install Windows (or Linux) as normal onto a known-good PC.
This OS installation is then tweaked as desired ready to be rolled out across the other PCs to be set up: any necessary
programs are installed, colours etc. are customised, and specialist drivers may be installed. Note that, in the case of
Windows, the "fast startup" setting must be disabled, as this prevents the capture process from being executed properly.
Once the OS is installed and customised, the computer should be shut down (_not_ put to "sleep" or "hibernate").

Before the image can be captured from the PC, an image listing must be created in the FOG web interface and assigned to
the PC. Log into the FOG web configuration panel and select the _images_ page. Next, select the option to create a new
image and fill in the OS parameters as appropriate. Lastly, navigate to the _hosts_ page, select the correct PC, and
"assign" the aforementioned image to it. FOG now associates the registered PC with the newly-created image.

To start the image capture process, log into the FOG web interface (see above), select the computer from the "hosts" list,
and navigate to the "tasks" tab. Click the option to capture an image, and the task will be initiated. Next, boot up the
PC, once again selecting the "onboard NIC" boot option in the BIOS. This time, instead of showing a multiple-choice PXE
menu, the image capture process is immediately commenced, resulting in a _Partclone_ progress bar. Partclone is the image
capture software leveraged by FOG (see http://www.partclone.org). Partclone will capture the OS image and reboot the PC
when completed.

To verify that the image capture completed correctly, log into the server's FOG web interface and load up the images list.
A file size should now have appeared next to the OS image associated with this "golden image" PC, indicating that the
server correctly retrieved a copy of the image.

Deploying an image from the PXE server onto a PC:
-------------------------------------------------

Note that this process may be completed regardless of whether or not the target computer is registered to the deployment
server.

Once the "golden image" has been captured, it may be deployed to other computers. This is done by connecting the target
machine(s) to the local network on the client router (see Aerohive details in _System Components_ section) and
_PXE booting_ it by selecting the "onboard NIC" boot option in the computer BIOS upon power-up. The PXE boot retrieves
a minimal menu-based "OS" over the network from the deployment server. From this menu, select the "Deploy Image" option
and enter the FOG web interface credentials (see _System Components_ section above) when prompted. The image to be
deployed may then be selected from a list. Once an image is selected, the PXE OS uses Partclone (see preceding
section) to copy it across from the server onto the computers' local hard drive / SSD.

PXE booting basics:
-------------------

PXE stands for _Preboot Execution Environment_ and is a method by which a computer can boot into an operating system
_over a network connection_ rather than from a local disk or memory stick. It is noteworthy that this requires
a computer BIOS to suppport this functionality. Thankfully, the vast majority of modern BIOSes support PXE owing to its
widespread use as a tool for deploying operating systems on a large scale.

The particular way in which PXE is employed by the FOG deployment server involves a client PC booting into a minimal
"OS" entirely over a network connection. This minimal OS then handles the transfer of the main operating system image
(Windows/Linux etc.) onto the computer's main internal drive.

In general, booting into a PXE OS involves three stages:

1. The BIOS of the client PC establishes an Ethernet connection using the computer's on-board network card (NIC).
   The BIOS next sends out a _DHCP request_ to the router on the local Ethernet network asking for an IP address.
   DHCP stands for _Dynamic Host Configuration Protocol_ and is the process used by computers for obtaining IP
   addresses automatically from a central router.
   Notably, inside this request is included a couple of DHCP _option flag_ - identified by the numbers 66 and 67 -
   which instruct the router that this DHCP request originates from a computer looking for a PXE server. The
   router responds with an IP address for the client to use, but also includes specific responses for the option
   flags: the IP address of the _PXE server_ and the _boot filename_ which the PC should retrieve from the server
   to use as its boot material.
   
2. Once the client PC has obtained a DHCP IP address from the router, together with details of the PXE server and
   boot filename, the computer's BIOS uses TFTP to fetch the boot file from the PXE server. TFTP
   stands for _Trivial File Transfer Protocol_ and is preferable for low-level network booting applications
   owing to its simplicity.

3. With the boot file obtained from the server, the client PC boots this file

OpenVPN router-server connection details:
-----------------------------------------
