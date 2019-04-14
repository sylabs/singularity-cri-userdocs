.. _sykube:

======
Sykube
======

Imagine that you want to test your k8s cluster with Singularity CRI locally, but don't want to waste
time setting is all up or messing with Minikube.

*Sykube* comes to the rescue! Inspired by `Minikube <https://kubernetes.io/docs/setup/minikube/>`_, it allows
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


.. warning::
	Make sure you don't have any restrictions applied to iptables `FORWARD` target. To check this
	do the following:

	.. code-block:: bash

		$ sudo iptables -nL
		Chain FORWARD (policy DROP)
		target     prot opt source               destination
		DOCKER-USER  all  --  0.0.0.0/0            0.0.0.0/0
		DOCKER-ISOLATION-STAGE-1  all  --  0.0.0.0/0            0.0.0.0/0
		ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
		DOCKER     all  --  0.0.0.0/0            0.0.0.0/0
		ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
		ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0

	Problems are often caused by Docker daemon since it adds custom	iptables rules.
	That prevents Sykube instance network from being correctly setup.

	To disable docker and reboot you should do the following:

	.. code-block:: bash

		$ sudo service docker stop && sudo systemctl disable docker && reboot

	After that Sykube should work correctly. Note this workaround may be redundant soon as
	there is an open GitHub issue referencing it `here <https://github.com/containernetworking/plugins/pull/75>`_.


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

