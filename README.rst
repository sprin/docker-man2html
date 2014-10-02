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

   docker run -it --name man2html_ctr sprin/centos7:manpages /bin/bash

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

   docker commit man2html_ctr man2html_img


Let's see it in the list:

.. code::

   docker images

Now let's generate some nice manpages with that new image!

.. code::

   docker run man2html /bin/bash -c '/usr/bin/gunzip -c $(man -w systemd) | /usr/bin/man2html' > /tmp/tmp.html && open /tmp/tmp.html

Good old Readability
====================

http://www.tobez.org/download/readability.html

Dockerfiles
===========

.. code::

   git clone git@github.com:sprin/docker-man2html.git
   cd docker-man2html
   docker build -t man2html .

Now we have an image identical to the first one, but created automatically
using the definition in the Dockerfile.

Daemons in my Dockers
=====================

Let's run an nginx daemon:

.. code::

   docker run -d -p 80 --name nginx nginx

This will expose port 80 of the nginx container on a random port, which
you can see with `docker ps`.

Extra Credit!
=============

Try linking the man2html container to the nginx container so nginx can serve
the generated HTML files.
