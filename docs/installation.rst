.. _installation:

============
Installation
============

This document will guide you through the process of installing Singularity-CRI on a Linux host.

--------
Overview
--------

Singularity-CRI is nothing more than Singularity-specific implementation of `Kubernetes CRI
<https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md>`_.

It is currently under development and passes 71/74
`validation tests <https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/validation.md>`_.
Note that used test suite is taken from `v1.13.0` tag. Detailed report can be found
`here <https://docs.google.com/spreadsheets/d/1Ym3K4LddqKNc4LCh8jr5flN7YDxfnM_hrLxpeDJRO1k/edit?usp=sharing>`_.

----------------
Before you begin
----------------

If you have an earlier version of Singularity-CRI installed, you should :ref:`remove
it <remove-an-old-version>` before executing the installation commands.  You
will also need to install some dependencies as described below.


.. _install-dependencies:

--------------------
Install Dependencies
--------------------

1) Install `git <https://git-scm.com/downloads>`_
2) Install `Singularity 3.1+ <https://www.sylabs.io/guides/3.0/user-guide/installation.html>`_ with OCI support
3) Install `Go 1.11+ <https://golang.org/doc/install>`_
4) Install `inotify <http://man7.org/linux/man-pages/man7/inotify.7.html>`_ to enable GPU device plugin
5) Install socat if you want to enable fort-forwarding, e.g

.. code-block:: bash

    sudo apt-get install socat

--------------------
Install from source
--------------------

The following commands will install Singularity-CRI from the `GitHub repo
<https://github.com/sylabs/singularity-cri>`_  to the ``/usr/local/bin``.

The ``master`` branch contains the latest, bleeding edge version of Singularity-CRI.
This is the default branch when you clone the source code, so you don't have to check out any new branches
to install it. The ``master`` branch changes quickly and may be unstable. Thus installing from tag is the
preferred way and is described below.

Since Singularity-CRI is now built with `go modules <https://github.com/golang/go/wiki/Modules>`_
there is no need to create standard `go workspace <https://golang.org/doc/code.html>`_.
If you still prefer keeping source code under ``GOPATH`` make sure ``GO111MODULE=on`` is set.

The following assumes you want set up Singularity-CRI outside ``GOPATH``.

.. code-block:: bash

	git clone https://github.com/sylabs/singularity-cri.git && \
	cd singularity-cri && \
	git checkout tags/v1.0.0-beta.2 -b v1.0.0-beta.2 && \
	make && \
	sudo make install

After these commands Singularity-CRI will be installed in the ``/usr/local/bin`` directory.

Refer to :ref:`configuration section <configuration>` to see how Singularity-CRI can be configured.

---------------
Important notes
---------------

Because images external to the Library are in a format other than SIF, when pulled they are converted to this native
format for use by Singularity. Each time a SIF file is created through this conversion process a timestamp is
automatically generated and captured as SIF metadata. Unfortunately, changes in the timestamp result in uniquely
tagged images - even though the only difference is the timestamp in the SIF metadata. This matter has been classified
as a known issue for documentation; refer to `issue <https://github.com/sylabs/singularity-cri/issues/15>`_
for additional details.

.. _remove-an-old-version:

---------------------
Remove an old version
---------------------

When you run ``sudo make install``, the command lists files as they are
installed. They must be removed in order to completely remove Singularity-CRI from your host.
For convenience we created uninstall command, so you can run the following to cleanup installation:

.. code-block:: bash

    sudo make uninstall
