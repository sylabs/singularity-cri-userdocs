.. _installation:

============
Installation
============

.. _sec:installation:

This document will guide you through the process of installing Singularity CRI on existing
Kubernetes **v1.12+** cluster. If you don't have Kubernetes cluster already set up, please reference
`official installation guide <https://kubernetes.io/docs/setup/pick-right-solution/#bare-metal>`_.
Further assumed Linux environment since it is the only operating system that can support containers because of
kernel features like namespaces

If you are looking for trying Singularity CRI locally, follow :ref:`local testing guide with Sykube <sykube>`.

--------
Overview
--------

Singularity CRI is nothing more than Singularity-specific implementation of `Kubernetes CRI
<https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md>`_.
It is currently under development and passes 70/74 `validation tests
<https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/validation.md>`_.
Detailed report can be found
`here <https://docs.google.com/spreadsheets/d/1Ym3K4LddqKNc4LCh8jr5flN7YDxfnM_hrLxpeDJRO1k/edit?usp=sharing>`_.

To fully enable Singularity support in Kubernetes cluster Singularity CRI should be installed
on each Kubernetes node. Further you may find installation steps for a single node.

----------------
Before you begin
----------------

If you have an earlier version of Singularity CRI installed, you should :ref:`remove
it <remove-an-old-version>` before executing the installation commands.  You
will also need to install some dependencies as described below.

.. _install-dependencies:


--------------------
Install Dependencies
--------------------

1) Install `git <https://git-scm.com/downloads>`_
2) Install `Singularity 3.1+ <https://www.sylabs.io/guides/3.0/user-guide/installation.html>`_ with OCI support
3) Install `Go 1.11+ <https://golang.org/doc/install>`_
4) Install socat, e.g

.. code-block:: bash

    $ sudo apt-get install socat

--------------------
Install from source
--------------------

The following commands will install Singularity from the `GitHub repo
<https://github.com/sylabs/singularity-cri>`_  **master branch** to ``/usr/local``.

The ``master`` branch contains the latest, bleeding edge version of Singularity CRI.
This is the default branch when you clone the source code, so you don't have to check out any new branches
to install it. The ``master`` branch changes quickly and may be unstable.

Download Singularity CRI repo
=================================

Since Singularity-CRI is now build with `go modules <https://github.com/golang/go/wiki/Modules>`_
there in no need to create standard `go workspace <https://golang.org/doc/code.html>`_. If you still
prefer keeping source code under $GOPATH make sure GO111MODULE is set.

The following assumes you want to set up Singularity CRI outside $GOPATH.
To set up project do the following:

.. code-block:: bash

    $ git clone https://github.com/sylabs/singularity-cri.git && \
	cd singularity-cri && \
	make dep


Build and install Singularity CRI
=================================

Assuming you are in repository root directory:

.. code-block:: bash

	$ make && sudo make install

After this command Singularity CRI will be installed in the ``/usr/local`` directory hierarchy.

Configuration
-------------

Singularity CRI can be configured before the startup with a YAML config file.
Example configuration can be found `here <https://github.com/sylabs/singularity-cri/blob/master/config/sycri.yaml>`_.

Upon installation this default ``sycri.yaml`` config is copied to ``/usr/local/etc/sycri/sycri.yaml`` and that is
the default location of the config Singularity CRI will look for. To override this behavior one can
specify ``-config`` flag with path to the desired config file, e.g:

.. code-block:: bash

	$ sudo sycri -config ~/my-config.yaml

It is also possible to change log level with ``-v`` flag, e.g:

.. code-block:: bash

	$ sudo sycri -v 10


Configure node to use Singularity CRI
=====================================

To make Kubernetes work with Singularity CRI a couple of steps are needed:

1) create Singularity CRI service
2) modify kubelet config
3) restart kubelet with new config


Create Singularity CRI service
------------------------------

To create the systemd service do the following:

.. code-block:: bash

	$ cat > /etc/systemd/system/sycri.service <<EOF
		[Unit]
		Description=Singularity CRI
		After=network.target
		StartLimitIntervalSec=0

		[Service]
		Type=simple
		Restart=always
		RestartSec=1
		ExecStart=/usr/local/bin/sycri -v 10
		Environment="PATH=/usr/local/libexec/singularity/bin:/bin:/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"

		[Install]
		WantedBy=multi-user.target
	  EOF

	$ sudo systemctl enable sycri && \
	  sudo systemctl start sycri


To verify Singularity CRI is running do the following:

.. code-block:: bash

	$ sudo systemctl status sycri

You should see the following output:

.. code-block:: text

	● sycri.service - Singularity CRI
	   Loaded: loaded (/etc/systemd/system/sycri.service; enabled; vendor preset: enabled)
	   Active: active (running) since Fri 2019-02-22 15:59:02 UTC; 2min 54s ago
	 Main PID: 31927 (sycri)
		Tasks: 9 (limit: 4915)
	   CGroup: /system.slice/sycri.service
			   └─31927 /usr/local/bin/sycri -v 10

	Feb 22 15:59:02 kube01 systemd[1]: Started Singularity CRI.
	Feb 22 15:59:02 kube01 sycri[31927]: I0222 15:59:02.061441   31927 network.go:112] Network configuration found: bridge
	Feb 22 15:59:02 kube01 sycri[31927]: I0222 15:59:02.061598   31927 main.go:102] Singularity CRI server started on /var/run/singularity.sock

Optionally you may want to disable other runtime services, e.g. docker daemon.

Modify kubelet config
---------------------

Kubelet need to be reconfigured so that it connects to Singularity CRI. If you haven't change default config,
the following will be enough:

.. code-block:: bash

    $ cat > /etc/default/kubelet <<EOF
		KUBELET_EXTRA_ARGS=--container-runtime=remote --container-runtime-endpoint=/var/run/singularity.sock --image-service-endpoint=/var/run/singularity.sock
	  EOF

If you have changed ``listenSocket`` make sure you pass it to kubelet as well.


Restart kubelet service
-----------------------

.. code-block:: bash

	$ sudo systemctl restart kubelet


That's it! After you completed those steps for each node, consider your cluster configured
to use Singularity as a container runtime. For examples refer to :ref:`basic usage section <basic_usage>`.

.. _remove-an-old-version:

---------------------
Remove an old version
---------------------

When you run ``sudo make install``, the command lists files as they are
installed. They must all be removed in order to completely remove Singularity CRI.
For convenience we created uninstall command, so you can run the following to cleanup installation:

.. code-block:: bash

    $ sudo make uninstall
