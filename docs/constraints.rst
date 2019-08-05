.. _constraints:

=============================
Known issues and constraints
=============================

---------------------------------------------
Differentiate same image with different tags
---------------------------------------------

Because images external to the Library are in a format other than SIF, when pulled they are converted to this native
format for use by Singularity. Each time a SIF file is created through this conversion process a timestamp is
automatically generated and captured as SIF metadata. Unfortunately, changes in the timestamp result in uniquely
tagged images - even though the only difference is the timestamp in the SIF metadata. This matter has been classified
as a known issue for documentation; refer to `issue <https://github.com/sylabs/singularity-cri/issues/15>`_
for additional details.


---------------------------------
Using image from private registry
---------------------------------

Unfortunately, it is not possible to use *on-prem* Sylabs library or pull private SIF images when working with
Singularity-CRI. However, this is expected to change soon as we work with Kubernetes maintainers on
the `issue <https://github.com/kubernetes/kubernetes/issues/79803>`_. As a current workaround we suggest
to configure each node individually.

It is still possible to pull private SIF images from `Cloud Library <cloud.sylabs.io>`_ using
`image pull secrets <https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/>`_.
This will require creating a secret of type `kubernetes.io/dockerconfigjson` (a `proposal
<https://github.com/kubernetes/enhancements/pull/1171>`_ is already created in order to change this hardcoded
part of Kubernetes). The full flow is the following:

1. Create an access token to the Cloud Library (see `docs <https://sylabs.io/guides/3.3/user-guide/cloud_library.html?highlight=token#creating-a-access-token>`_)

2. Create pull secret

.. code-block:: bash

	$ kubectl create secret docker-registry cloud-secret \
	  --docker-server=cloud.sylabs.io \
	  --docker-username=<any-name-here> \
	  --docker-password=<cloud-token>

3. Use pull secret during pod creation

.. code-block:: yaml

	apiVersion: v1
	kind: Pod
	metadata:
	  name: secret
	spec:
	  containers:
	    - name: secret
	      image: cloud.sylabs.io/sashayakovtseva/default/secret:interactive
	      tty: true
	      stdin: true
	  imagePullSecrets:
	    - name: cloud-secret
