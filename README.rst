=========================
OpenShift SDN devel docs
=========================

Developer documentation for OpenShift SDN team

This document intends to be a quick devref and guide for new developers. It'll
be a WIP covering details such as how to build, run, configure and test features.

This mainly covers three different repos:

.. tabs ::

  .. tab :: Cluster Network Operator

    The Cluster Network Operator installs and upgrades the networking
    components on an OpenShift Kubernetes cluster.

  .. tab :: OpenShift SDN

    OpenShift SDN is the default network plugin for OpenShift. It uses Open
    vSwitch to connect pods locally, with VXLAN tunnels to connect different nodes.

  .. tab :: OVN-Kubernetes

    Kubernetes integration for OVN (Open Virtual Network) is a system to
    support virtual network abstraction.
