.. _scheduler:

==============================
Extending Kubernetes Scheduler
==============================

Since SIF images do not support multiarch builds, its arch should be respected properly
when it comes to pod scheduling. There are two options to make sure your SIF-based pod is
scheduled on the node with a matching architecture: using `nodeSelector` in pod spec
or configuring arch-scheduler extender.

This document will guide you through the second approach.
To extend Kubernetes with arch extender the following steps are needed:

#. launch arch-scheduler pod
#. modify Kubernetes scheduler config
#. restart Kubernetes scheduler

Launch arch-scheduler pod
-------------------------

Clone arch-scheduler repo:

.. code-block:: bash

	$ git clone https://github.com/sylabs/arch-scheduler

Launch arch-scheduler pod:

.. code-block:: bash

	$ cd arch-scheduler && \
	  kubectl apply -f deploy/extender.yaml


This will make arch-scheduler run on the same node where default Kubernetes scheduler is located.
Location of arch-scheduler is important since later Kubernetes scheduler will query it on localhost.

Modify Kubernetes scheduler config
----------------------------------

This step requires changing current scheduler policy to include arch-scheduler extender in extenders list.

If you have already modified scheduler policy you probably know how to do that. Further assumed default
scheduler has not been altered and arch-scheduler extender is the first change applied.

Copy config and policy files under default Kubernetes directory:

.. code-block:: bash

	$ sudo mkdir /etc/kubernetes/scheduler && \
	  sudo cp deploy/config.yaml /etc/kubernetes/scheduler && \
	  sudo cp deploy/policy.yaml /etc/kubernetes/scheduler

Modify kube-scheduler pod:

.. code-block:: bash

	$ sudo cp deploy/scheduler.yaml /etc/kubernetes/manifests/kube-scheduler.yaml

.. note::

	`/etc/kubernetes/manifests/kube-scheduler.yaml` is the default location of
	scheduler pod specification. If you changed that make sure to update the appropriate one.

Restart kube-scheduler pod
---------------------------

By default Kubernetes watches for any changes on
`static pods <https://kubernetes.io/docs/tasks/administer-cluster/static-pod/>`_, and default scheduler is one of them.
This means right after you updated kube-scheduler pod changes should automatically apply in some
reasonable amount of time. At the end you should see all system pods running without any issues:

.. code-block:: bash

	$ kubectl get po --namespace=kube-system


However, if kube-scheduler is not in a `Running` state, try to simply delete it and let
Kubernetes recreate it once again correctly:

.. code-block:: bash

	$ kubectl delete po --namespace=kube-system kube-scheduler

.. note::

	Kube-scheduler pod name may vary, make sure you are using correct one when deleting pod.
