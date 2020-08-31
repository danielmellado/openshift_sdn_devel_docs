Fedora 32 setup for KIND
------------------------

Change to use cgroupsv1 instead of cgroupsv2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* https://gist.github.com/linuxkathirvel/ef6a368bf88946416319986e53628e43

.. code-block::

   sudo vim /etc/default/grub

Add `systemd.unified_cgroup_hierarchy=0` in kernel command line. It will look like below
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`GRUB_CMDLINE_LINUX="resume=/dev/mapper/vv-swap rd.lvm.lv=vv/root rd.lvm.lv=vv/swap rhgb quiet intel_iommu=on systemd.unified_cgroup_hierarchy=0"`

Run below command after add it. The below `grub.cfg` location is only of UEFI.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning::
   Don't use this for Legacy BIOS.

.. code-block:: shell-session

   sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
   sudo reboot

Verify
~~~~~~

* run `sudo virt-host-validate` to check. If all the items are `PASS`, it is
  working fine. `Fuse` module may `FAIL`. It should be load manually.

.. code-block:: shell-session

  sudo virt-host-validate
    QEMU: Checking for hardware virtualization                                 : PASS
    QEMU: Checking if device /dev/kvm exists                                   : PASS
    QEMU: Checking if device /dev/kvm is accessible                            : PASS
    QEMU: Checking if device /dev/vhost-net exists                             : PASS
    QEMU: Checking if device /dev/net/tun exists                               : PASS
    QEMU: Checking for cgroup 'cpu' controller support                         : PASS
    QEMU: Checking for cgroup 'cpuacct' controller support                     : PASS
    QEMU: Checking for cgroup 'cpuset' controller support                      : PASS
    QEMU: Checking for cgroup 'memory' controller support                      : PASS
    QEMU: Checking for cgroup 'devices' controller support                     : PASS
    QEMU: Checking for cgroup 'blkio' controller support                       : PASS
    QEMU: Checking for device assignment IOMMU support                         : PASS
    QEMU: Checking if IOMMU is enabled by kernel                               : PASS
     LXC: Checking for Linux >= 2.6.26                                         : PASS
     LXC: Checking for namespace ipc                                           : PASS
     LXC: Checking for namespace mnt                                           : PASS
     LXC: Checking for namespace pid                                           : PASS
     LXC: Checking for namespace uts                                           : PASS
     LXC: Checking for namespace net                                           : PASS
     LXC: Checking for namespace user                                          : PASS
     LXC: Checking for cgroup 'cpu' controller support                         : PASS
     LXC: Checking for cgroup 'cpuacct' controller support                     : PASS
     LXC: Checking for cgroup 'cpuset' controller support                      : PASS
     LXC: Checking for cgroup 'memory' controller support                      : PASS
     LXC: Checking for cgroup 'devices' controller support                     : PASS
     LXC: Checking for cgroup 'freezer' controller support                     : PASS
     LXC: Checking for cgroup 'blkio' controller support                       : PASS
     LXC: Checking if device /sys/fs/fuse/connections exists                   : PASS


Change firewalld to use iptables instead of nftables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* https://kind.sigs.k8s.io/docs/user/known-issues/#fedora32-firewalld

This sed script does not work, do it manually. Todo: fix the sed…

.. code-block:: shell-session

  sed -i /etc/firewalld/firewalld.conf 's/FirewallBackend=.*/FirewallBackend=iptables/'
  systemctl restart firewalld

Install go
~~~~~~~~~~

.. code-block:: shell-session

  git clone https://github.com/udhos/update-golang.git
  cd update-golang
  ./update-golang

Set GOPATH in ~/.bashrc
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: shell-session

  export GOPATH=$HOME/go
  export PATH=$PATH:$GOPATH/bin
  export PATH=$PATH:/usr/local/go/bin
  export PATH=$PATH:$HOME/bin

Install KIND
~~~~~~~~~~~~

* https://kind.sigs.k8s.io/docs/user/quick-start/

.. code-block:: shell-session

   GO111MODULE="on" go get sigs.k8s.io/kind@v0.8.1
  [or… git clone https://github.com/kubernetes-sigs/kind.git; make; cp bin/kind /usr/bin/kind]

Install kubectl
~~~~~~~~~~~~~~~

* https://kubernetes.io/docs/tasks/tools/install-kubectl/

.. code-block:: shell-session

  curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

Install docker
~~~~~~~~~~~~~~

* https://fedoramagazine.org/docker-and-fedora-32/

.. code-block:: shell-session

  sudo firewall-cmd --permanent --zone=trusted --add-interface=docker0
  sudo firewall-cmd --permanent --zone=FedoraWorkstation --add-masquerade
  sudo dnf install moby-engine docker-compose
  sudo systemctl enable docker

Install ovn-kubernetes
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: shell-session

  git clone https://github.com/ovn-org/ovn-kubernetes.git

Install kubernetes
~~~~~~~~~~~~~~~~~~

.. code-block:: shell-session

  cd $GOPATH
  mkdir k8s.io; cd k8s.io
  git clone https://github.com/kubernetes/kubernetes.git

Build kubernetes e2e.test binary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: shell-session

  cd GOPATH/github.com/k8s.io/kubernetes
  make WHAT="test/e2e/e2e.test vendor/github.com/onsi/ginkgo/ginkgo"

Start KIND cluster
~~~~~~~~~~~~~~~~~~

.. code-block:: shell-session

  cd $GOPATH/github.com/ovn-org/ovn-kubernetes/contrib
  ./kind.sh

export KUBECONFIG=$HOME/admin.conf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: shell-session

  kubectl get nodes -A -o wide

  [root@nfvsdn-06 contrib]# kubectl get nodes -o wide
  NAME                STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION           CONTAINER-RUNTIME
  ovn-control-plane   Ready    master   5m58s   v1.18.2   172.18.0.4    <none>        Ubuntu 20.04 LTS   5.7.17-200.fc32.x86_64   containerd://1.3.3-14-g449e9269
  ovn-worker          Ready    <none>   5m22s   v1.18.2   172.18.0.3    <none>        Ubuntu 20.04 LTS   5.7.17-200.fc32.x86_64   containerd://1.3.3-14-g449e9269
  ovn-worker2         Ready    <none>   5m25s   v1.18.2   172.18.0.2    <none>        Ubuntu 20.04 LTS   5.7.17-200.fc32.x86_64   containerd://1.3.3-14-g449e9269

Run all network policy tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: shell-session

  cd $GOPATH/k8s.io./kubernetes
  ./_output/local/go/bin/e2e.test -kubeconfig  $HOME/admin.conf -ginkgo.focus="\[sig-network\].*Policy" -num-nodes 2

Run a specific network policy test
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: shell-session

  cd $GOPATH/k8s.io./kubernetes
  ./_output/local/go/bin/e2e.test -kubeconfig  $HOME/admin.conf -ginkgo.focus="should stop enforcing policies after they are deleted"
