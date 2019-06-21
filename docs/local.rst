.. _local:

=============
Local testing
=============

Further you may find a way to smoketest Singularity-CRI without Kubernetes installation.

----------------------
Setting up environment
----------------------

1. Install `crictl <https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md>`_:

.. code-block:: bash

	VERSION="v1.12.0" && \
	wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz && \
	sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin && \
	rm -f crictl-$VERSION-linux-amd64.tar.gz

2. Configure it work with Singularity-CRI. Create `/etc/crictl.yaml` config file and add the following:

.. code-block:: text

	runtime-endpoint: unix:///var/run/singularity.sock
	image-endpoint: unix:///var/run/singularity.sock
	timeout: 10
	debug: false

For details on all available options see `crictl install page
<https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md#install-crictl>`_.

3. Build and launch Singularity-CRI server (optionally: :ref:`configure <configuration>`):

.. code-block:: bash

	make clean && \
	make && \
	sudo make install && \
	sudo sycri


-----------------------------
Interact with Singularity-CRI
-----------------------------

We will walk through basic examples of interaction with Singularity-CRI running.
Further assumed you are located in Singularity-CRI repository root.

Running Nginx
-------------

1. Run a pod which exposes port 80:

.. code-block:: bash

	sudo crictl runp examples/net-pod.json

That will return you ID of a freshly created pod. You will also see it when listing all pods on host:

.. code-block:: bash

	sudo crictl pods

2. Create & start nginx container inside the just created pod:

.. code-block:: bash

	sudo crictl pull nginx && \
	sudo crictl create <POD_ID> examples/nginx.json examples/net-pod.json

Here ``POD_ID`` is the ID of a pod you want your container to appear in. If everything is fine
you will get ID of a created container. Also you will see container when listing all containers on host:

.. code-block:: bash

	sudo crictl ps -a

Unlike pods, containers should be explicitly started after creation:

.. code-block:: bash

	sudo crictl start <CONTAINER_ID>


Verify Nginx container is running by opening `localhost:80 <http://localhost:80>`_ in any browser.
You should see the Nginx welcome page.


Running info container
----------------------

1. Run any pod (we will use the same pod from the previous example):

.. code-block:: bash

	sudo crictl runp examples/net-pod.json

2. Create & start container that outputs some system info:

.. code-block:: bash

	sudo crictl pull cloud.sylabs.io/sashayakovtseva/test/test-info && \
	sudo crictl create <POD_ID> examples/info-cont.json examples/net-pod.json && \
	sudo crictl start <CONTAINER_ID>

Verify container executed correctly by opening logs:

.. code-block:: bash

	sudo crictl logs <CONTAINER_ID>


The expected output is something like the following:

.. code-block:: text

	args: [./test]
	mounts: 602 548 0:57 / / rw,relatime - overlay overlay rw,lowerdir=/var/run/singularity/containers/fa96e2cdaec1081a8b229fe2d8f64ac80b698b7a07f303629fb60b36abbeec8e/bundle/rootfs,upperdir=/var/run/singularity/containers/fa96e2cdaec1081a8b229fe2d8f64ac80b698b7a07f303629fb60b36abbeec8e/bundle/overlay/upper,workdir=/var/run/singularity/containers/fa96e2cdaec1081a8b229fe2d8f64ac80b698b7a07f303629fb60b36abbeec8e/bundle/overlay/work
	603 602 0:50 / /proc rw,nosuid,nodev,noexec,relatime - proc proc rw
	604 602 0:59 / /dev rw,nosuid - tmpfs tmpfs rw,size=65536k,mode=755
	605 604 0:60 / /dev/pts rw,nosuid,noexec,relatime - devpts devpts rw,gid=5,mode=620,ptmxmode=666
	606 604 0:61 / /dev/shm rw,nosuid,nodev,noexec,relatime - tmpfs shm rw,size=65536k
	607 604 0:49 / /dev/mqueue rw,nosuid,nodev,noexec,relatime - mqueue mqueue rw
	608 602 0:56 / /sys ro,nosuid,nodev,noexec,relatime - sysfs sysfs ro
	609 602 0:22 /singularity/pods/85d02f45ee7fdf05aa199abafad6b1617fd018b3aacf30883c4724ebb025dac2/hostname /etc/hostname ro,relatime shared:5 - tmpfs tmpfs rw,size=403956k,mode=755
	610 602 8:1 /var/lib/singularity /mounted1 ro,relatime - ext4 /dev/sda1 rw,errors=remount-ro,data=ordered
	hostname: networking <nil>
	pwd: / <nil>
	content of /
		     Lrwxrwxrwx        0	.exec -> .singularity.d/actions/exec
		     Lrwxrwxrwx        0	.run -> .singularity.d/actions/run
		     Lrwxrwxrwx        0	.shell -> .singularity.d/actions/shell
		     drwxr-xr-x        0	.singularity.d ->
		     Lrwxrwxrwx        0	.test -> .singularity.d/actions/test
		     drwxr-xr-x        0	bin ->
		     drwxr-xr-x        0	dev ->
		     Lrwxrwxrwx        0	environment -> .singularity.d/env/90-environment.sh
		     drwxr-xr-x        0	etc ->
		     drwxr-xr-x        0	home ->
		     drwxr-xr-x        0	lib ->
		     drwxr-xr-x        0	media ->
		     drwxr-xr-x        0	mnt ->
		     drwxr-xr-x        0	mounted1 ->
		     dr-xr-xr-x        0	proc ->
		     drwx------        0	root ->
		     drwxr-xr-x        0	run ->
		     drwxr-xr-x        0	sbin ->
		     Lrwxrwxrwx        0	singularity -> .singularity.d/runscript
		     drwxr-xr-x        0	srv ->
		     dr-xr-xr-x        0	sys ->
		     -rwxr-xr-x        0	test ->
		     dtrwxr-xr-x       0	tmp ->
		     drwxr-xr-x        0	usr ->
		     drwxr-xr-x        0	var ->
	uid=0 gid=0 euid=0 egid=0
	pid=30 ppid=0
	envs=[LD_LIBRARY_PATH=/.singularity.d/libs SHLVL=1 MY_ANOTHER_VAR=is-awesome PS1=Singularity>  TERM=xterm PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin PWD=/ MY_CUSTOM_VAR=singularity-cri]
	...


Cleanup examples
----------------

The quickest way to cleanup is simply by removing containing pods:

.. code-block:: bash

	sudo crictl stopp <POD_ID> && \
	sudo crictl rmp <POD_ID>
