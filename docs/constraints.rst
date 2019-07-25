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

Unfortunately, it is not possible to use on-prem Sylabs library or pull private SIF images when working with
Singularity-CRI. However, this is expected to change soon as we work with Kubernetes maintainers on
the `issue <https://github.com/kubernetes/kubernetes/issues/79803>`_. As a current workaround we suggest
to configure each node individually.
