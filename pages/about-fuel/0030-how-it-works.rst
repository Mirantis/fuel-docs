.. raw:: pdf

   PageBreak

.. index: How Fuel Works

.. _How-Fuel-Works:

How Fuel Works
==============

In practice, Fuel works as follows:

1. First, set up Fuel Master Node using the ISO. This process only needs to 
   be completed once per installation.

2. Next, power on your virtual or physical nodes to make them discoverable by
   Fuel Master Node and configure your OpenStack environment using the Fuel UI or CLI.

3. Finally, deploy your OpenStack environment on discovered nodes. Fuel will 
   perform all verification and deployment magic for you.

Fuel is designed to enable you to maintain your environment while giving you the 
flexibility to adapt it to your own configuration.

.. image:: /_images/how-it-works_svg.jpg
  :align: center

Fuel comes with several pre-defined deployment configurations, some of them 
include additional configuration options that allow you to adapt OpenStack 
deployment to your environment.

Fuel UI integrates all of the deployments scripts into a unified, 
Web-based Graphical User Interface that walks administrators through the 
process of installing and configuring a fully functional OpenStack environment.
