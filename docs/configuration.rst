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

It is also possible to change logging level with ``-v`` flag.
The higher the level, the more verbose logs you will see:

.. code-block:: bash

	$ sycri -v 10

