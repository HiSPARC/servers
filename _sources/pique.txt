Installation of Pique
=====================

Granting davidf rights to manage software and services:

.. code-block:: sh

    (root)$ visudo

and adding::

    davidf  ALL = SOFTWARE, SERVICES

Preparing for source install:

.. code-block:: sh

    (root)$ cd /localstore
    (root)$ mkdir -p usr/local
    (root)$ mv /usr/local/src usr/local
    (root)$ cd /usr/local
    (root)$ ln -s /localstore/usr/local/src .
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


Public database web application
-------------------------------

The public database blablabla.
It is a pure python implementation under complete version control.

Prerequisites
^^^^^^^^^^^^^

The public database application uses PyTables and the underlying HDF5
library to read binary data files.  PyTables depends heavily on NumPy.:

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

The public databases graphing capabilities come from Enthought Chaco, a
python plotting library.  It needs swig to build.  Install with:

.. code-block:: sh

    $ wget http://prdownloads.sourceforge.net/swig/swig-1.3.40.tar.gz
    $ tar xvzf swig-1.3.40.tar.gz
    $ cd swig-1.3.40
    $ ./configure
    $ make
    (root)$ make install

It also needs a GUI toolkit, like wxPython:

.. code-block:: sh

    $ wget http://downloads.sourceforge.net/wxpython/wxPython-src-2.8.10.1.tar.bz2
    $ tar xvjf wxPython-src-2.8.10.1.tar.bz2
    $ cd wxPython-src-2.8.10.1
    $ ./configure --enable-unicode --with-opengl
    $ make && make -C contrib/src/gizmos && make -C contrib/src/stc
    (root)$ make install && make -C contrib/src/gizmos install && make -C contrib/src/stc install
    $ cd wxPython/src/gtk
    $ patch < /usr/local/src/hisparc/gdi.patch
    $ cd ../..
    (root)$ python setup.py install

The contents of the aforementioned gdi.patch is::

    --- wxPython/src/gtk/_gdi_wrap.cpp.orig 2009-08-08 16:26:48.000000000 +0200
    +++ wxPython/src/gtk/_gdi_wrap.cpp      2009-08-08 16:32:50.000000000 +0200
    @@ -4195,6 +4195,10 @@
         virtual wxGraphicsBrush CreateRadialGradientBrush(wxDouble , wxDouble , wxDouble , wxDouble , wxDouble ,
                                                           const wxColour &, const wxColour &)  { return wxNullGraphicsBrush; }
         virtual wxGraphicsFont CreateFont( const wxFont & , const wxColour & ) { return wxNullGraphicsFont; }
    +
    +    // patch required as explained in
    +    // http://groups.google.com/group/wxPython-users/browse_thread/thread/129ba27e2f868c3c?pli=1
    +    wxGraphicsBitmap CreateBitmap( const wxBitmap &bitmap ) const { return wxNullGraphicsBitmap; } 
     };

We currently run Chaco straight out of the subversion repository.  Once a
new release has been finalized, we might go back to simply install from
PyPI.  Now, however, we have to issue:

.. code-block:: sh

    (root)$ easy_install etsprojecttools
    $ ets co Chaco
    (root)$ ets install Chaco_3.2.1


Setting up the public database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In summary:

Here we go:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ bzr co sftp://admhispa@login.nikhef.nl/project/hisparc/bzr/publicdb/trunk publicdb
    (root)$ cd /var/www
    (root)$ mkdir django_publicdb
    (root)$ chown davidf.hisparc django_publicdb/
    $ ln -s /usr/local/src/hisparc/publicdb/django_publicdb/* .
    $ cp --remove-destination /usr/local/src/hisparc/publicdb/django_publicdb/settings.py .
    $ cp --remove-destination /usr/local/src/hisparc/publicdb/django_publicdb/manage.py .
    $ cp /usr/local/src/hisparc/publicdb/examples/django.wsgi .

And edit django.wsgi so that it contains the right system path::

    sys.path.append('/var/www')

Then, added the public database to the Apache configuration:

.. code-block:: sh

    (root)$ cd /etc/httpd/conf.d/
    (root)$ touch hisparc.conf
    (root)$ chown davidf.hisparc hisparc.conf 
    (root)$ chmod g+w hisparc.conf 

And edit hisparc.conf to contain::

    RedirectMatch ^/$ http://data.hisparc.nl/django/

    WSGIScriptAlias /django /var/www/django_publicdb/django.wsgi
    WSGIPythonEggs /tmp

    Alias /django/media /usr/local/lib/python2.6/site-packages/Django-1.1.1-py2.6.egg/django/contrib/admin/media

Reload Apache configuration:

.. code-block:: sh

    $ sudo /sbin/service httpd reload


TODO
----

South.


.. code-block:: sh

    mkdir /var/www/media
    chown www.www media
    ln -s /var/www/django_publicdb/static media

    cd /usr/local/bin
    cp /usr/local/src/hisparc/publicdb/examples/django-cron.py hisparc-update

    # Run a daily check for new events, but it _must_ be a few hours after
    # midnight, so don't place this script in cron.daily, just to be sure.
    0 4 * * * root /usr/local/bin/hisparc-update

    python PIL

    django cron script on pique, changed a bit?
