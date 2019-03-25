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

YAML configuration that we will be using is the following:

.. code-block:: yaml

	apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	  name: image-service-deployment
	  namespace: default
	spec:
	  replicas: 1
	  template:
	    metadata:
	      labels:
	    	app: image-service
	      name: image-service
	      namespace: default
	    spec:
	      containers:
	      - name: image-server
	        image: cloud.sylabs.io/sashayakovtseva/test/image-server
	        ports:
	        - containerPort: 8080
	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: image-service
	spec:
	  type: NodePort
	  ports:
	    - port: 80
	      targetPort: 8080
	  selector:
	    app: image-service


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
