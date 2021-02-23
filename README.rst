=========================
OpenShift SDN devel docs
=========================

Developer documentation for OpenShift SDN team

This document intends to be a quick devref and guide for new developers. It'll
be a WIP covering details such as how to build, run, configure, and test features.

This mainly covers three different repos:

.. tabs ::

  .. tab :: Cluster Network Operator

    The Cluster Network Operator installs and upgrades the networking
    components on an OpenShift Kubernetes cluster.

  .. tab :: OpenShift SDN

    OpenShift SDN is the default network plug-in for OpenShift. It uses Open
    vSwitch (OVS) to connect pods locally and uses VXLAN tunnels to connect different nodes.

  .. tab :: OVN-Kubernetes

    OVN-Kubernetes is a network plug-in for Kubernetes clusters and is not
    unique to OpenShift. This network plug-in provides Kubernetes integration
    for Open Virtual Network (OVN). OVN provides virtual network abstraction
    and uses OVS on each host to connect pods locally. Generic Network
    Virtualization Encapsulation (Geneve) provides network tunneling to
    connect different nodes.
