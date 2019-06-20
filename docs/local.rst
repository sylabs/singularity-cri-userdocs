.. _local:

=============
Local testing
=============


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

For details on all options available see `crictl install page
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


4. In separate terminal run nginx pod:

.. code-block:: bash

$ cd examples

$ sudo crictl runp net-pod.json
0e0538d57a52d8673b9ad5124dd017087b3f5292f82cb9406c10f270f8f531fa

$ sudo crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT
0e0538d57a52d       26 seconds ago      Ready               networking          default             1


5. Then create & start nginx container inside the just created pod:

.. code-block:: bash

$ sudo crictl pull nginx

# sudo crictl create <podID> nginx.json net-pod.json
$ sudo crictl create 0e0538d57a52d nginx.json net-pod.json
7a83219a135ebb79133bac065d861e174488ba81a6622a10e2ec7e8b5b1b4371

$ sudo crictl ps -a
CONTAINER ID        IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID
7a83219a135eb       nginx               4 seconds ago       Created             nginx-container     1                   0e0538d57a52d

# sudo crictl start <containerID>
$ sudo crictl start 7a83219a135eb
7a83219a135eb


Verify nginx container is running by openning [localhost:80](http://localhost:80) in any browser.
You should see nginx welcome page.

6. You can also run container that outputs some system info (to smoke test CRI):

.. code-block:: bash

$ sudo crictl pull cloud.sylabs.io/sashayakovtseva/test/test-info

# sudo crictl create <podID> info-cont.json net-pod.json
$ sudo crictl create 0e0538d57a52d info-cont.json net-pod.json
bf040d311ca7d929ee20de4973df5c00aaf6f0e733feb695e985757686fb121b

$ sudo crictl ps -a
CONTAINER ID        IMAGE                                    		  	CREATED             STATE               NAME                ATTEMPT             POD ID
bf040d311ca7d       cloud.sylabs.io/sashayakovtseva/test/test-info   	10 seconds ago      Created             testcontainer       1                   0e0538d57a52d

# sudo crictl start <containerID>
$ sudo crictl start bf040d311ca7d
bf040d311ca7d


Verify container executed correctly by opening logs:


 .. code-block:: bash

# sudo crictl logs <containerID>
$ sudo crictl logs bf040d311ca7d


The expected output is the following:

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
	    dtrwxr-xr-x        0	tmp ->
	     drwxr-xr-x        0	usr ->
	     drwxr-xr-x        0	var ->
uid=0 gid=0 euid=0 egid=0
pid=30 ppid=0
envs=[LD_LIBRARY_PATH=/.singularity.d/libs SHLVL=1 MY_ANOTHER_VAR=is-awesome PS1=Singularity>  TERM=xterm PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin PWD=/ MY_CUSTOM_VAR=singularity-cri]
...


7. Cleanup examples
	The quickest way to cleanup is simply pod removal:

.. code-block:: bash

# sudo crictl stopp <podID>
$ sudo crictl stopp 0e0538d57a52d

# sudo crictl rmp <podID>
$ sudo crictl rmp 0e0538d57a52d

