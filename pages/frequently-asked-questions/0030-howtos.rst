.. raw:: pdf

   PageBreak

HowTo Notes
===========

.. index:: HowTo: Create the XFS partition

.. _create-the-XFS-partition:

HowTo: Create the XFS partition
-------------------------------

In most cases, Fuel creates the XFS partition for you.  If for some reason you 
need to create it yourself, use this procedure:

1. Create the partition itself::

  fdisk /dev/sdb
    n(for new)
    p(for partition)
    <enter> (to accept the defaults)
    <enter> (to accept the defaults)
    w(to save changes)

2. Initialize the XFS partition::

  mkfs.xfs -i size=1024 -f /dev/sdb1

3. For a standard swift install, all data drives are mounted directly under 
   /srv/node, so first create the mount point::

  mkdir -p /srv/node/sdb1

4. Finally, add the new partition to fstab so it mounts automatically, then 
   mount all current partitions::

  echo "/dev/sdb1 /srv/node/sdb1 xfs
  noatime,nodiratime,nobarrier,logbufs=8 0 0" >> /etc/fstab
  mount -a

.. index:: HowTo: Redeploy a node from scratch

.. _Redeploy_node_from_scratch:
    
HowTo: Redeploy a node from scratch
------------------------------------

Compute and Cinder nodes in an HA configuration and controller in any 
configuration cannot be redeployed without completely redeploying the cluster.  
However, in a non-HA situation you can redeploy a Compute or Cinder node.  
To do so, follow these steps:

1. Remove the certificate for the node by executing the command     
   ``puppet cert clean <hostname>`` on Fuel Master node.
2. Reboot the node over the network so it can be picked up by cobbler.
3. Run the puppet agent on the target node using ``puppet agent --test``.

.. _Enable_Disable_Galera_autorebuild:

.. index:: HowTo: Galera Cluster Autorebuild

HowTo: Enable/Disable Galera Cluster Autorebuild Mechanism
----------------------------------------------------------

By defaults Fuel reassembles Galera cluster automatically without need for any 
user interaction.

To prevent `autorebuild feature` you shall do::

  crm_attribute -t crm_config --name mysqlprimaryinit --delete

To re-enable `autorebuild feature` you should do::
  
  crm_attribute -t crm_config --name mysqlprimaryinit --update done

.. index:: HowTo: Troubleshoot Corosync/Pacemaker

How To Troubleshoot Corosync/Pacemaker
--------------------------------------

Pacemaker and Corosync come with several CLI utilities that can help you 
troubleshoot and understand what is going on.

crm - Cluster Resource Manager
++++++++++++++++++++++++++++++

This is the main pacemaker utility it shows you state of pacemaker cluster. 
Several most popular commands that you can use to understand whether your 
cluster is consistent:

**crm status**

This command shows you the main information about pacemaker cluster and state of 
resources being managed::

  crm(live)# status
  ============
  Last updated: Tue May 14 15:13:47 2013
  Last change: Mon May 13 18:36:56 2013 via cibadmin on fuel-controller-01
  Stack: openais
  Current DC: fuel-controller-01 - partition with quorum
  Version: 1.1.6-9971ebba4494012a93c03b40a2c58ec0eb60f50c
  5 Nodes configured, 5 expected votes
  3 Resources configured.
  ============

  Online: [ fuel-controller-01 fuel-controller-02 fuel-controller-03 
  fuel-controller-04 fuel-controller-05 ]

  p_quantum-plugin-openvswitch-agent (ocf::pacemaker:quantum-agent-ovs): Started fuel-controller-01
  p_quantum-dhcp-agent (ocf::pacemaker:quantum-agent-dhcp): Started fuel-controller-01
  p_quantum-l3-agent (ocf::pacemaker:quantum-agent-l3): Started fuel-controller-01

**crm(live)# resource**

Here you can enter resource-specific commands::

  crm(live)resource#  status`

  p_quantum-plugin-openvswitch-agent  (ocf::pacemaker:quantum-agent-ovs) Started 
  p_quantum-dhcp-agent   (ocf::pacemaker:quantum-agent-dhcp) Started 
  p_quantum-l3-agent     (ocf::pacemaker:quantum-agent-l3) Started 

**crm(live)resource#  start|restart|stop|cleanup <resource_name>**

These commands let you correspondingly start, stop, restart resources. 

**cleanup**

Cleanup command cleans resources state on the nodes in case of their failure or 
unexpected operation, e.g. some residuals of SysVInit operation on resource, in 
which case pacemaker will manage it by itself, thus deciding in which node to 
run the resource.

E.g.::

  3 Nodes configured, 3 expected votes
  3 Resources configured.
  ============

  3 Nodes configured, 3 expected votes
  16 Resources configured.


  Online: [ controller-01 controller-02 controller-03 ]

   vip__management_old	(ocf::heartbeat:IPaddr2):   	Started controller-01
   vip__public_old    	(ocf::heartbeat:IPaddr2):   	Started controller-02
   Clone Set: clone_p_haproxy [p_haproxy]
    Started: [ controller-01 controller-02 controller-03 ]
   Clone Set: clone_p_mysql [p_mysql]
    Started: [ controller-01 controller-02 controller-03 ]
   Clone Set: clone_p_quantum-openvswitch-agent [p_quantum-openvswitch-agent]
    Started: [ controller-01 controller-02 controller-03 ]
   Clone Set: clone_p_quantum-metadata-agent [p_quantum-metadata-agent]
    Started: [ controller-01 controller-02 controller-03 ]
   p_quantum-dhcp-agent   (ocf::mirantis:quantum-agent-dhcp): 	Started controller-01
   p_quantum-l3-agent 	(ocf::mirantis:quantum-agent-l3):   	Started controller-03

In this case there were residual OpenStack agent processes that were started by 
pacemaker in case of network failure and cluster partitioning. After the 
restoration of connectivity pacemaker saw these duplicate resources running on 
different nodes. You can let it clean up this situation automatically or, if you 
do not want to wait, cleanup them manually.

.. seealso:: 

  crm interactive help and documentation resources for Pacemaker 
  (e.g. http://doc.opensuse.org/products/draft/SLE-HA/SLE-ha-guide_sd_draft/cha.ha.manual_config.html).

In some network scenarios one can get cluster split into several parts and 
``crm status`` showing something like this::

  On ctrl1
  ============
  ….
  Online: [ ctrl1 ]

  On ctrl2
  ============
  ….
  Online: [ ctrl2 ]

  On ctrl3
  ============
  ….
  Online: [ ctrl3 ]

You can troubleshoot this by checking corosync connectivity between nodes. 
There are several points:

1) Multicast should be enabled in the network, IP address configured as 
   multicast should not be filtered, mcastport and mcasport - 1 udp ports should 
   be accepted on management network between controllers

2) corosync should start after network interfaces are configured

3) `bindnetaddr` should be in the management network or at least in the same 
   multicast reachable segment

You can check this in output of ``ip maddr show``:

.. code-block:: none
   :emphasize-lines: 1,8
  
   5:  br-mgmt
      link  33:33:00:00:00:01
      link  01:00:5e:00:00:01
      link  33:33:ff:a3:e2:57
      link  01:00:5e:01:01:02
      link  01:00:5e:00:00:12
      inet  224.0.0.18
      inet  239.1.1.2
      inet  224.0.0.1
      inet6 ff02::1:ffa3:e257
      inet6 ff02::1

**corosync-objctl**

This command is used to get/set runtime corosync configuration values including 
status of corosync redundant ring members::

  runtime.totem.pg.mrp.srp.members.134245130.ip=r(0) ip(10.107.0.8) 
  runtime.totem.pg.mrp.srp.members.134245130.join_count=1
  ...
  runtime.totem.pg.mrp.srp.members.201353994.ip=r(0) ip(10.107.0.12) 
  runtime.totem.pg.mrp.srp.members.201353994.join_count=1
  runtime.totem.pg.mrp.srp.members.201353994.status=joined


If IP of the node is 127.0.0.1 it means that corosync started when only loopback 
interfaces was available and bound to it.

If there is only one IP in members list that means there is corosync connectivity 
issue because the node does not see the other ones. The same stays for the case 
when members list is incomplete.

.. index:: HowTo: Smoke Test HA

How To Smoke Test HA
--------------------

To test if Quantum HA is working, simply shut down the node hosting, e.g. Quantum 
agents (either gracefully or hardly). You should see agents start on the other node::


  # crm status

  Online: [ fuel-controller-02 fuel-controller-03 fuel-controller-04 fuel-controller-05 ]
  OFFLINE: [ fuel-controller-01 ]

  p_quantum-plugin-openvswitch-agent (ocf::pacemaker:quantum-agent-ovs): Started fuel-controller-02
  p_quantum-dhcp-agent (ocf::pacemaker:quantum-agent-dhcp): Started fuel-controller-02
  p_quantum-l3-agent (ocf::pacemaker:quantum-agent-l3): Started fuel-controller-02

and see corresponding Quantum interfaces on the new Quantum node::

  # ip link show

  11: tap7b4ded0e-cb: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc 
  12: qr-829736b7-34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc 
  13: qg-814b8c84-8f: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc 

You can also check ``ovs-vsctl show output`` to see that all corresponding 
tunnels/bridges/interfaces are created and connected properly::

  ce754a73-a1c4-4099-b51b-8b839f10291c
    Bridge br-mgmt
        Port br-mgmt
            Interface br-mgmt
                type: internal
        Port "eth1"
            Interface "eth1"
    Bridge br-ex
        Port br-ex
            Interface br-ex
                type: internal
        Port "eth0"
            Interface "eth0"
        Port "qg-814b8c84-8f"
            Interface "qg-814b8c84-8f"
                type: internal
    Bridge br-int
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port br-int
            Interface br-int
                type: internal
        Port "tap7b4ded0e-cb"
            tag: 1
            Interface "tap7b4ded0e-cb"
                type: internal
        Port "qr-829736b7-34"
            tag: 1
            Interface "qr-829736b7-34"
                type: internal
    Bridge br-tun
        Port "gre-1"
            Interface "gre-1"
                type: gre
                options: {in_key=flow, out_key=flow, remote_ip="10.107.0.8"}
        Port "gre-2"
            Interface "gre-2"
                type: gre
                options: {in_key=flow, out_key=flow, remote_ip="10.107.0.5"}
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port "gre-3"
            Interface "gre-3"
                type: gre
                options: {in_key=flow, out_key=flow, remote_ip="10.107.0.6"}
        Port "gre-4"
            Interface "gre-4"
                type: gre
                options: {in_key=flow, out_key=flow, remote_ip="10.107.0.7"}
        Port br-tun
            Interface br-tun
                type: internal
    ovs_version: "1.4.0+build0"
 
