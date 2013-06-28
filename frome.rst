Installation of Frome
=====================

Granting davidf rights to manage software and services:

.. code-block:: sh

    (root)$ visudo

and adding::

    davidf  ALL = SOFTWARE, SERVICES

Preparing for source install:

.. code-block:: sh

    (root)$ cd /usr/local/src/
    (root)$ mkdir hisparc
    (root)$ chown davidf.hisparc hisparc/
    $ chmod g+w hisparc/

In /etc/ld.so.conf.d new file usrlocal.conf, to let ldconfig find
libraries of locally installed software::

    /usr/local/lib


Python
------

Python:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ wget http://www.python.org/ftp/python/2.6.4/Python-2.6.4.tgz
    $ tar xvzf Python-2.6.4.tgz
    $ cd Python-2.6.4
    $ ./configure --enable-shared
    $ make
    (root)$ make install

Then, run:

.. code-block:: sh

    (root)$ ldconfig

Now, the python libraries are registered.


Python Setuptools
-----------------

From egg:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ wget http://pypi.python.org/packages/2.6/s/setuptools/setuptools-0.6c11-py2.6.egg#md5=bfa92100bd772d5a213eedd356d64086
    (root)$ sh setuptools-0.6c11-py2.6.egg 


IPython, an interactive Python shell
------------------------------------

Download and install IPython:

.. code-block:: sh

    (root)$ easy_install ipython


Web server
----------

Install apache development libraries:

.. code-block:: sh

    $ sudo yum install httpd-devel

    ================================================================================
     Package             Arch        Version                 Repository        Size
    ================================================================================
    Installing:
     httpd-devel         i386        2.2.3-31.sl5.2          sl-security      147 k
     httpd-devel         x86_64      2.2.3-31.sl5.2          sl-security      147 k
    Installing for dependencies:
     apr                 x86_64      1.2.7-11.el5_3.1        sl-security      118 k
     apr-devel           x86_64      1.2.7-11.el5_3.1        sl-security      237 k
     apr-util            x86_64      1.2.7-7.el5_3.2         sl-security       74 k
     apr-util-devel      x86_64      1.2.7-7.el5_3.2         sl-security       53 k
     httpd               x86_64      2.2.3-31.sl5.2          sl-security      1.2 M

Change configuration in /etc/httpd/conf/httpd.conf. Patch::

    --- httpd.conf.orig     2009-12-04 14:35:39.000000000 +0100
    +++ httpd.conf  2009-12-04 14:35:50.000000000 +0100
    @@ -228,8 +228,8 @@
     #  when the value of (unsigned)Group is above 60000; 
     #  don't use Group #-1 on these systems!
     #
    -User apache
    -Group apache
    +User www
    +Group www
     
     ### Section 2: 'Main' server configuration
     #

Enabling httpd on startup:

.. code-block:: sh

    $ sudo /sbin/chkconfig --add httpd
    $ sudo /sbin/chkconfig --levels 35 httpd on

Starting httpd now:

.. code-block:: sh

    $ sudo /sbin/service httpd start

For mod_wsgi:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ wget http://modwsgi.googlecode.com/files/mod_wsgi-3.1.tar.gz
    $ tar xvzf mod_wsgi-3.1.tar.gz 
    $ cd mod_wsgi-3.1
    $ ./configure
    $ make
    (root)$ make install

Change configuration in /etc/httpd/conf/httpd.conf. Patch::

    --- httpd.conf.orig     2009-12-04 15:19:01.000000000 +0100
    +++ httpd.conf  2009-12-04 15:34:30.000000000 +0100
    @@ -197,6 +197,7 @@
     LoadModule mem_cache_module modules/mod_mem_cache.so
     LoadModule cgi_module modules/mod_cgi.so
     LoadModule version_module modules/mod_version.so
    +LoadModule wsgi_module modules/mod_wsgi.so
     
     #
     # The following modules are not loaded by default:

Restarting apache:

.. code-block:: sh

    $ sudo /sbin/service httpd restart


Version control
---------------

Install bazaar from source:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ wget http://launchpad.net/bzr/2.0/2.0.2/+download/bzr-2.0.2.tar.gz
    $ tar xvzf bzr-2.0.2.tar.gz
    $ cd bzr-2.0.2
    (root)$ python setup.py install


Paramiko
^^^^^^^^

Paramiko supports ssh2 for python, which is needed to do a checkout of our
application's sources over sftp.  Install using easy_install:

.. code-block:: sh

    (root)$ easy_install paramiko

This will automatically download, compile and install dependencies
(pycrypto).


Datastore web application
-------------------------

The datastore application is driving our central data storage solution.
It is a pure python implementation under complete version control.

Prerequisites
^^^^^^^^^^^^^

The datastore application uses PyTables and the underlying HDF5 library to
store binary data files.  PyTables depends heavily on NumPy.:

.. code-block:: sh

    (root)$ easy_install numpy

This gives an error::

    /tmp/easy_install-JePGOA/numpy-1.4.0rc1/numpy/distutils/misc_util.py:248: RuntimeWarning: Parent module 'numpy.distutils' not found while handling absolute import
    Error in atexit._run_exitfuncs:
    Traceback (most recent call last):
      File "/usr/local/lib/python2.6/atexit.py", line 24, in _run_exitfuncs
        func(*targs, **kargs)
      File "/tmp/easy_install-JePGOA/numpy-1.4.0rc1/numpy/distutils/misc_util.py", line 248, in clean_up_temporary_directory
    ImportError: No module named numpy.distutils
    Error in sys.exitfunc:
    Traceback (most recent call last):
      File "/usr/local/lib/python2.6/atexit.py", line 24, in _run_exitfuncs
        func(*targs, **kargs)
      File "/tmp/easy_install-JePGOA/numpy-1.4.0rc1/numpy/distutils/misc_util.py", line 248, in clean_up_temporary_directory
    ImportError: No module named numpy.distutils

So, rerun the command, this time without errors:

.. code-block:: sh

    (root)$ easy_install numpy

Now:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ wget http://www.hdfgroup.org/ftp/HDF5/prev-releases/hdf5-1.8.3/src/hdf5-1.8.3.tar.gz
    $ tar xvzf hdf5-1.8.3.tar.gz 
    $ cd hdf5-1.8.3
    $ ./configure --prefix=/usr/local
    $ make
    (root)$ make install
    (root)$ ldconfig

And, finally, install PyTables itself:

.. code-block:: sh

    (root)$ easy_install tables


Setting up datastore
^^^^^^^^^^^^^^^^^^^^

In summary:

    - Created a /var/www/wsgi-bin directory from which to run the wsgi
      applications
    - Created a subdirectory owned by davidf.hisparc inside this wsgi-bin
    - Did a checkout of the datastore sources inside the subdirectory
    - Made a local copy of the application into the parent (wsgi-bin) and
      edited to set the correct local full path
    - Added the wsgi application to the Apache configuration

Here we go:

.. code-block:: sh

    (root)$ cd /var/www
    (root)$ mkdir wsgi-bin
    (root)$ cd wsgi-bin
    (root)$ mkdir datastore
    (root)$ chown davidf.hisparc datastore
    (root)$ chmod g+w datastore
    $ cd /var/www/wsgi-bin/datastore/
    $ bzr co sftp://admhispa@login.nikhef.nl/project/hisparc/bzr/datastore/trunk .

Copy the application.wsgi and config.ini from the examples directory:

.. code-block:: sh

    (root)$ cd /var/www/wsgi-bin
    (root)$ cp datastore/examples/application.wsgi datastore.wsgi
    (root)$ cp datastore/examples/config.ini datastore/
    (root)$ chown davidf.hisparc datastore.wsgi datastore/config.ini
    (root)$ chmod g+w datastore.wsgi datastore/config.ini

Edited /var/www/wsgi-bin/datastore.wsgi and set the correct paths::

    sys.path.append('/var/www/wsgi-bin/datastore/wsgi')
    configfile = ('/var/www/wsgi-bin/datastore/config.ini')

The config.ini now reads::

    [General]
    log=/var/log/hisparc/hisparc.log
    loglevel=debug
    station_list=/databases/frome/station_list.csv
    data_dir=/databases/frome

    [Writer]
    sleep=1

I had to create the appropriate directory in /var/log and grant rights:

.. code-block:: sh

    (root)$ cd /var/log
    (root)$ mkdir hisparc
    (root)$ chown www.hisparc hisparc
    (root)$ chmod g+w hisparc

Then, added datastore to the Apache configuration:

.. code-block:: sh

    (root)$ cd /etc/httpd/conf.d/
    (root)$ touch hisparc.conf
    (root)$ chown davidf.hisparc hisparc.conf 
    (root)$ chmod g+w hisparc.conf 

And edited hisparc.conf to contain::

    WSGIScriptAlias /hisparc/upload /var/www/wsgi-bin/datastore.wsgi

Reload Apache configuration:

.. code-block:: sh

    $ sudo /sbin/service httpd reload


TODO
----

Writer app!


(Maybe) Not relevant
--------------------

install: yum-utils
easy_install dozer
easy_install pil (requirement of dozer)
easy_install mysql-python (for migration)
install: gcc-gfortran
easy_install virtualenvwrapper
install: blas-devel lapack-devel (for scipy)
