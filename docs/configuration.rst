.. _configuration:

=============
Configuration
=============

Singularity-CRI can be configured before the startup with a YAML config file.
Example configuration can be found
`here <https://github.com/sylabs/singularity-cri/blob/master/config/sycri.yaml>`_.

Upon installation this default ``sycri.yaml`` config is copied to ``/usr/local/etc/sycri/sycri.yaml`` and
that is the default location of the config file Singularity-CRI will look for. To override this behavior
one can specify ``--config`` flag with path to the desired config file:

.. code-block:: bash

	$ sycri --config ~/my-config.yaml

It is also possible to change logging level with ``-v`` flag. Singularity-CRI follows Kubernetes
`logging convention <https://github.com/kubernetes/community/blob/master/contributors/devel/sig-instrumentation/logging.md>`_

Additionally you may specify log level 6 to enable Singularity runtime debug logging:

.. code-block:: bash

	$ sycri -v 6

