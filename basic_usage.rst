.. _basic_usage:

===========
Basic Usage
===========

When Singularity CRI is installed and configured and kubelet is restarted,
you may use k8s as you usually do. Here we will show some basic k8s usage so you can
verify your installation. Full list of examples can be found `in Singularity CRI repo
<https://github.com/sylabs/singularity-cri/tree/master/examples/k8s>`_.

------------
Hello, cats!
------------

Here we will walk through a basic k8s example with SIF. We will deploy http file server
that listens on port 8080 and create a k8s service to make it public on port 80.

YAML configuration that we will be using is located
`here <https://github.com/sylabs/singularity-cri/blob/master/examples/k8s/image-service.yaml>`_.

.. note::
	To make Singularity CRI pull image from `cloud library <https://cloud.sylabs.io/library>`_ an explicit
	**cloud.sylabs.io** prefix should be specified in image field.


To create a deployment and a service run the following:

.. code-block:: bash

	$ kubectl apply -f image-service.yaml
	deployment.extensions/image-service-deployment created
	service/image-service created

To verify objects are indeed created you can do:

.. code-block:: bash

	$ kubectl get deploy
	$ kubectl get svc

If everything is fine you should be able to access the file server via NodePort service.

----------------------------------
Image recognition using NVIDIA GPU
----------------------------------

Here we will image recognition app that uses NVIDIA GPU.

.. image:: darkflow.png


YAML configuration that we will be using is located
`here <https://github.com/sylabs/singularity-cri/blob/master/examples/k8s/gpu/darkflow.yaml>`_.

To create a deployment and a service run the following:

.. code-block:: bash

	$ kubectl apply -f darkflow.yaml
	configmap/web-config created
	deployment.extensions/darkflow created
	deployment.extensions/darkflow-front created
	deployment.extensions/darkflow-web created
	service/darkflow created
	service/darkflow-front created
	service/darkflow-web created

To verify objects are indeed created you can do:

.. code-block:: bash

	$ kubectl get deploy
	$ kubectl get svc

If everything is fine you should be able to access Darkflow UI that is exposed with darkflow-web service.

.. note::
	You may need to change serverURL value in config map from the example above according to
	your cluster configuration. Also you can change `input` and `output` directories
	location on your host.
