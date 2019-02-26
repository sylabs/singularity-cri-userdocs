.. _sykube:

======
Sykube
======

Imagine that you want to test your k8s cluster with Singularity CRI locally, but don't want to waste
time setting is all up or messing with Minikube.

*Sykube* comes to the rescue! Inspired by `Minikube <https://kubernetes.io/docs/setup/minikube/)>`_, it allows
you to run k8s cluster locally within **Singularity instances** with Singularity CRI baked in. Moreover, unlike
Minikube, it is capable of spawning not only one, but arbitrary amount of nodes.

Another nice feature is *ephemeral* clusters. With this option on, Sykube will create local cluster
on *tmpfs* making sure nothing is left after host reboot.

.. image:: sykube-cluster.png


------------
Installation
------------

Assuming you already have Singularity 3.1+ installed, do the following:

.. code-block:: bash

	$ sudo singularity run library://sykube

This with pull the Sykube image and add a binary in `/usr/local/bin`. To verify your installation
you can check usage options with the following command:

.. code-block:: bash

	$ sykube

---------------------
Running local cluster
---------------------

Before creating new Sykube cluster make sure you :ref:`removed any existing <clean-up>`.
To create new Sykube cluster do the following:

.. code-block:: bash

	$ sykube init


This may take a few minutes, stay patient.

After that if you already have `kubectl <https://kubernetes.io/docs/tasks/tools/install-kubectl/>`_ installed, you
may want to configure it to work with Sykube cluster. To do that run the following:

.. code-block:: bash

	$ sykube config > ~/.kube/config

If you don't have kubectl, you can use Sykube, e.g:

.. code-block:: bash

	$ sykube exec master kubectl <args>


.. _clean-up:

-----------
Cleaning up
-----------

After testing you may want to remove the cluster. To do that run the following:

.. code-block:: bash

	$ sykube stop && sykube delete

