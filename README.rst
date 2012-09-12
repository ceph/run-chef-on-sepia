===================
 Run Chef on Sepia
===================

Here are quick & ugly scripts to run Chef on Sepia Vercoi/Senta.


Installation
============

You need a VPN connection to the Sepia lab, and you need your SSH
public key authorized.

Install Downburst_ and set it up. The ``downburst`` command must be in
your ``PATH``; for example, symlink
``~/src/downburst/virtualenv/bin/downburst`` into ``~/bin/downburst``
and add ``PATH="$PATH:$HOME/bin"`` in your ``.bashrc``.

.. _Downburst: https://github.com/ceph/downburst

Add aliases to avoid typing so much::

    install -d -m0755 ~/.libvirt
    cat >~/.libvirt/libvirt.conf <<-'EOF'
	uri_aliases = [
	    'vercoi01=qemu+ssh://ubuntu@vercoi01.front.sepia.ceph.com/system?no_tty',
	    'vercoi02=qemu+ssh://ubuntu@vercoi02.front.sepia.ceph.com/system?no_tty',
	    'vercoi03=qemu+ssh://ubuntu@vercoi03.front.sepia.ceph.com/system?no_tty',
	    'vercoi04=qemu+ssh://ubuntu@vercoi04.front.sepia.ceph.com/system?no_tty',
	    'vercoi05=qemu+ssh://ubuntu@vercoi05.front.sepia.ceph.com/system?no_tty',
	    'vercoi06=qemu+ssh://ubuntu@vercoi06.front.sepia.ceph.com/system?no_tty',
	    'vercoi07=qemu+ssh://ubuntu@vercoi07.front.sepia.ceph.com/system?no_tty',
	    'vercoi08=qemu+ssh://ubuntu@vercoi08.front.sepia.ceph.com/system?no_tty',
            'senta01=qemu+ssh://ubuntu@senta01.front.sepia.ceph.com/system?no_tty',
            'senta02=qemu+ssh://ubuntu@senta02.front.sepia.ceph.com/system?no_tty',
            'senta03=qemu+ssh://ubuntu@senta03.front.sepia.ceph.com/system?no_tty',
            'senta04=qemu+ssh://ubuntu@senta04.front.sepia.ceph.com/system?no_tty',
	    ]
    EOF

Test your libvirt client::

    virsh -c vercoi08 hostname

Clone this repository (if you didn't already do that)::

    git clone https://github.com/ceph/run-chef-on-sepia.git
    cd run-chef-on-sepia

Whenever you want to run one of the included commands, ``cd`` into
this directory first.

Check that you have the needed dependencies installed. If not, do as
the output recommends::

   ./check-dependencies

Also, to run ``knife`` operations, you can either always ssh to the
chef server vm, or install Knife (from Chef) and `knife block`_.
(TODO this is not optional right now because we run
``knife block use`` automatically -- install it!)

.. _`knife block`: https://github.com/greenandsecure/knife-block/



Picking a server to use
=======================

There are several servers. You need to pick one. There is currently no
automation to help you pick.

The senta servers are the most recently deployed machines and likely
to be less loaded.

Avoid ``vercoi01`` and ``vercoi02``, they will be running more
production-ish vms.

Avoid servers that are low on free RAM. Use ``virsh -c vercoiNN
nodememstats``.

You can spread a single Chef setup over multiple servers.
(TODO the config file style isn't helpful, will fix.)


Usage
=====

Things you need:

- ``URI``: the libvirt server to talk to, looks like ``vercoiNN``
- ``NAME``: pick a unique name for your run -- it should include your
  username in it, for example ``jdoe-bug1234``

For one test run, you'll keep using the same ``NAME``. Different
``NAME`` values are used to separate different users from each other,
and you can use it to work on multiple setups at the same time.

There is no need to include things like ``chef`` in ``NAME``; the
virtual machines will name ``chef-`` in their name, and they will be
identified as ``server`` or ``node`` as appropriate.


TODO document config creation, or migrate to cli

To create the server node::

    ./server-create URI NAME

And wait for the installation to complete. Use ``virt-manager`` to see
the console and manage the virtual machines; install it with ``sudo
apt-get install virt-manager``.

(TODO make that more interactive, wait until completion)

Optionally, if you want to run ``knife`` locally::

    ./server-setup-knife NAME

-----

Once the server is running, add nodes by running::

    ./node-create URI NAME NODENAME
    ./node-create URI NAME ANOTHERNODE

The nodes will register with the Chef server automatically, as seen in
``knife node list``.

At this time, the node will not have any data disks. To later add
them, run::

    ./node-add-disk URI NAME NODENAME

A typical ``NODENAME`` might just be a running sequence number.

After you've installed Ceph using Chef roles, to make Ceph use the
disk, ssh in to the node and run::

    ssh ubuntu@chef-NAME-node-NODENAME
    sudo ceph-disk-prepare /dev/vdXXX
