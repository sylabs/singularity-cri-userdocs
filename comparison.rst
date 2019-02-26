.. _comparison:

=====================================
Comparing Singularity CRI with others
=====================================

You may wonder, what makes Singularity CRI different from other CRI implementations.

Well, there are couple of reasons. First of all, that is the only implementation fully
compatible and designed specially for Singularity.

Singularity is known for its security and performance, especially when it comes to HPC.
Unlike other popular runtimes, Singularity is **not** run as a daemon on a node, which prevents
lots of security leaks.

Secondly, Singularity CRI makes use of SIF images, which allows you to use all SIF benefits out of the box.
For instance, Singularity CRI will automatically check SIF signatures upon pulling an image. Also all pulled
images that are not in SIF format will be automatically converted to SIF.
