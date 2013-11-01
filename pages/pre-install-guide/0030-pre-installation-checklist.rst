.. index:: Pre-Installation Checklist
.. index:: Pre-Installation Checklist

.. _Pre-installChecklist:

Pre-Installation Checklist
==========================

Careful preparation before you begin will save time later. 
To help plan your OpenStack deployment, you can use the following
checklist to quickly identify and confirm the basic steps necessary for
a successful deployment.

+--------------------------------------------------------+--------------+
| Item to confirm                                        | Status [tick |
|                                                        | if done]     |
+========================================================+==============+
| The Fuel server has an IPMI or out of band management  |              |
| system and you have access to it.                      |              |
+--------------------------------------------------------+--------------+
| The Fuel server hardware is able to boot the Fuel ISO  |              |
| from DVD, USB flash drive, or IPMI remote media.       |              |
+--------------------------------------------------------+--------------+
| The Fuel server has connectivity to all node servers   |              |
| through the L2 management network.                     |              |
+--------------------------------------------------------+--------------+
| A DHCP server is NOT installed on the management       |              |
| network. Fuel acts as a DHCP server for the node       |              |
| servers configured to network boot using PXE.          |              |
+--------------------------------------------------------+--------------+
| Spanning tree protocol (STP) is disabled on switch     |              |
| ports connected to the node servers’ ports.            |              |
+--------------------------------------------------------+--------------+
| There is NO configured tagged VLAN on a switch for the |              |
| administrative network (to the boot server from PXE)   |              |
+--------------------------------------------------------+--------------+
| The node servers are set to network boot using PXE.    |              |
+--------------------------------------------------------+--------------+
| You have a method to reboot node servers (remotely or  |              |
| onsite).                                               |              |
+--------------------------------------------------------+--------------+
| The network equipment supports VLANs.                  |              |
+--------------------------------------------------------+--------------+
| It is possible to configure a tagged port on your      |              |
| switch/switches\*                                      |              |
+--------------------------------------------------------+--------------+
| You have permit rules on TCP ports 22 and 8000 on the  |              |
| firewall for the Fuel server’s IP address (to access   |              |
| the Fuel server from your PC).                         |              |
+--------------------------------------------------------+--------------+

.. note:: You needd this for VLAN Manager or Quantum VLAN mode, where tagged
  ports are required.

If you checked all the boxes in the above table, you are ready to deploy
Mirantis Openstack with Fuel.
