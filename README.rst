=========================================
Using Docker to Generate Pretty man pages
=========================================

Let's say we want to compare man pages for a particular package across
different distributions. We can use Docker as a quick and easy way to pull down
man pages for any Linux distro. We can take it one step further to set up
a Docker image whose job it is to generate HTML man pages for a particular
distro.

Make your own image with the docs
=================================

   yum install -y dosfstools pykickstart squashfs-tools system-config-keyboard
   rpm -ivh http://people.centos.org/arrfab/CentOS7/LiveMedia/RPMS/python-imgcreate-20.1-2.el7.x86_64.rpm
   git clone git@github.com:katzj/ami-creator.git
   yum install python-pip
   pip install ez_setup
   cd ami-creator
   python setup.py install

   cd
   git clone git@github.com:sprin/sig-cloud-instance-build.git
   cd sig-cloud-instance-build/docker
   ami-creator -c centos-7.ks

   # Assuming docker is running
   ./img2docker.sh centos-7-201409302151.img manpages

Manually Configuring an Image
=============================

Run a Centos 7 container, downloading first if not already locally available:

.. code::

   docker run -it --name man2html sprin/centos7:manpages /bin/bash

Inside the container, install man2html:

.. code::

   yum install -y man2html

Quit the container with Ctrl-D. Let's see the image we just pulled:

.. code::

   docker images

Now let's list all containers, including the one that we just exited:

.. code::

   docker ps -a

We can commit the changes we made to the container to a new image. The
first arg is the container name, and the second is the image name.

.. code::

   docker commit man2html man2html


Let's see it in the list:

.. code::

   docker images

Now let's generate some nice manpages with that new image!

.. code::

   docker run manpages /bin/bash -c '/usr/bin/gunzip -c $(man -w systemd) | /usr/bin/man2html' > /tmp/tmp.html && open /tmp/tmp.html
