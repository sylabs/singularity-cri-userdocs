.. _k8s:

===========================
Integrating with Kubernetes
===========================

This document will guide you through the process of integrating Singularity-CRI with existing
Kubernetes **v1.12+** cluster. If you don't have Kubernetes cluster already set up, please reference
`official installation guide <https://kubernetes.io/docs/setup/>`_.

If you are looking for trying Singularity-CRI locally, follow :ref:`local testing guide with Sykube <sykube>`.

To fully enable Singularity support in Kubernetes cluster, Singularity-CRI should be installed
on each Kubernetes node. However, one may choose to have a heterogeneous cluster with multiple container runtimes.
In that case only dedicated Kubernetes nodes should be integrated, and no changes should be done to the rest.

To make Kubernetes work with Singularity-CRI a couple of steps are needed:

#. create Singularity-CRI service
#. modify kubelet config
#. restart kubelet with new config

Create Singularity-CRI service
------------------------------

Create a systemd service with the following content:

.. code-block:: text

	[Unit]
	Description=Singularity-CRI
	After=network.target
	StartLimitIntervalSec=0

	[Service]
	Type=simple
	Restart=always
	RestartSec=1
	ExecStart=/usr/local/bin/sycri
	Environment="PATH=/usr/local/libexec/singularity/bin:/bin:/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"

	[Install]
	WantedBy=multi-user.target

Enable and start service afterwards:

.. code-block:: bash

	$ sudo systemctl enable sycri && \
	  sudo systemctl start sycri


.. note::
	Singularity-CRI service above uses default configuration and log level. You can modify both
	of them if you wish, refer to :ref:`configuration section <configuration>`.

.. note::

	Latest Singularity plugin system is not stable and leads to panic when no ``HOME`` and ``GOPATH``
	environments are set. There is an `open issue <https://github.com/sylabs/singularity/issues/3163>`_
	related to this problem, so until it is open or if you have a bugged version,
	you may need to add the following line to Singularity-CRI service definition:

	.. code-block:: text

		[Service]
		...
		Environment="GOPATH=/home/<your-name>/go"


To verify Singularity-CRI is running do the following:

.. code-block:: bash

	$ sudo systemctl status sycri

You should see the following output:

.. code-block:: text

	* sycri.service - Singularity-CRI
	   Loaded: loaded (/etc/systemd/system/sycri.service; enabled; vendor preset: enabled)
	   Active: active (running) since Fri 2019-02-22 15:59:02 UTC; 2min 54s ago
	 Main PID: 31927 (sycri)
		Tasks: 9 (limit: 4915)
	   CGroup: /system.slice/sycri.service
			   └─31927 /usr/local/bin/sycri

	Jun 20 16:01:38 ubuntu-bionic systemd[1]: Started Singularity-CRI.
	Jun 20 16:01:38 ubuntu-bionic sycri[1989]: I0620 16:01:38.980735    1989 network.go:112] Network configuration found: bridge
	Jun 20 16:01:38 ubuntu-bionic sycri[1989]: I0620 16:01:38.993158    1989 main.go:193] Singularity-CRI server started on /var/run/singularity.sock
	Jun 20 16:01:39 ubuntu-bionic sycri[1989]: I0620 16:01:39.095791    1989 device.go:97] Loading NVML
	Jun 20 16:01:39 ubuntu-bionic sycri[1989]: E0620 16:01:39.096779    1989 device.go:99] Could not initialize NVML library: could not load NVML library
	Jun 20 16:01:39 ubuntu-bionic sycri[1989]: W0620 16:01:39.097603    1989 main.go:209] GPU support is not enabled: unable to load: check libnvidia-ml.so.1 library and graphic drivers

.. note::

	We recommend disabling other runtime services, e.g. docker daemon.

Modify kubelet config
---------------------

Kubelet needs to be reconfigured so that it connects to Singularity-CRI.
If you haven't changed default config, the following will be enough:

.. code-block:: bash

	$ cat > /etc/default/kubelet <<EOF
	  KUBELET_EXTRA_ARGS=--container-runtime=remote \
	  --container-runtime-endpoint=unix:///var/run/singularity.sock \
	  --image-service-endpoint=unix:///var/run/singularity.sock
	  EOF

If you have changed ``listenSocket`` in Singularity-CRI configuration, make sure you pass that to kubelet
instead of a default `/var/run/singularity.sock`.


Restart kubelet service
-----------------------

.. code-block:: bash

	$ sudo systemctl restart kubelet


That's it! After you completed those steps for a node, consider it configured
to use Singularity as a container runtime. For examples refer to :ref:`examples section <examples>`.

GPU device plugin
-----------------

Singularity-CRI is shipped with built-in NVIDIA GPU device plugin. It will automatically
register itself in Kubernetes if node has any GPUs that can be discovered with NVML.

If GPU device-plugin was not enabled, you will see log line with the following content in Singularity-CRI logs,
and Singularity-CRI will continue serving requests as usual:

.. code-block:: text

	GPU support is not enabled: <reason>
