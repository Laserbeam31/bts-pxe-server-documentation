BTS FOG PXE Server Documentation
================================

Backstage has access to a _PC deployment server_. This is useful for deploying copies of Windows/Linux to PCs such as our thin
clients and Office computers. Unlike traditional OS installation from a DVD/USB, the deployment server allows PCs to have their
OSs installed over a network connection; this greatly facilitates OS installation on large numbers of machines.

The principle behind a deployment server is that a "golden image" of a desired OS (Operating System) is installed and customized
on one computer. Using the deployment server, this image is then _captured_ from the PC over the network connection using PXE
Preboot Execution Environment) booting. PXE is a process whereby the PC boots into a basic temporary OS over the network and uses
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

The FOG server features a web-based administration interface. This is accessible from: 

Registering a PC to the deployment server:
------------------------------------------

In order for the deployment server to be able to interface with a given computer, either for image capture or deployment,
the computer must be _registered_ with the server. This is a process whereby the computer's MAC address and system
hardware information is loaded into the FOG software's database.

To register the PC, it should be shut down and connected to the LAN (Local Area Network) connection on the
client Aerohive router. For more information on the Aerohive router's connectivity, see the _touring_aerohive_router_ repository.
Next, the PC should be _PXE booted_. This involves setting the PC's BIOS to use the "onboard NIC" boot menu option. With this
option, the PC will retrieve a minimal OS over the network from the deployment server. Note that this boot menu option
requires the PC's BIOS to be in _legacy boot mode_.

The minimal PXE OS results in a BTS-themed menu appearing on screen. The menu title clearly indicates whether the PC is
registered to the server. Assuming that the PC is not already registered, use the computer's keyboard to select the
"perform full host registration and inventory" option. Follow the various prompts to enter computer details; these
are stored on the PXE server's FOG database.

Capturing an image from a PC onto the deployment server:
--------------------------------------------------------

The first stage in preparing the "golden image" is to install Windows (or Linux) as normal onto a known-good PC.
This OS installation is then tweaked as desired ready to be rolled out across the other PCs to be set up: any necessary
programs are installed, colours etc. are customised, and specialist drivers may be installed. Note that, in the case of
Windows, the "fast startup" setting must be disabled, as this prevents the capture process from being executed properly.

Deploying an image from the PXE server onto a PC:
-------------------------------------------------

PXE booting basics:
-------------------

OpenVPN router-server connection details:
-----------------------------------------
