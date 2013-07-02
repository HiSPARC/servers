Upgrading of Nagios
===================

This describes how to update `Nagios Core <http://www.nagios.org>`_. This guide is about applying this update on tietar (:doc:`tietar`).


Current versions
----------------

Check current version of the OS, Apache and PHP.

.. code-block:: sh

    $ cat /etc/redhat-release
    CentOS release 5.7 (Final)
    $ httpd -v
    Server version: Apache/2.2.24
    $ php -v
    PHP 5.1.6
    $ yum list installed | grep php
    php.i386                  5.1.6-27.el5_5.3      installed
    php-cli.i386              5.1.6-27.el5_5.3      installed
    php-common.i386           5.1.6-27.el5_5.3      installed

Check the current version of Nagios and its packages.

.. code-block:: sh

    $ yum list installed | grep nagios
    nagios.i386               3.5.0-1.el5           installed
    nagios-nsca.i386          2.7.2-4.el5.rf        installed
    nagios-nsca-client.i386   2.7.2-4.el5.rf        installed
    nagios-plugins.i386       1.4.16-1.el5.rf       installed
    nagios-plugins-nrpe.i386  2.14-1.el5.rf         installed


Check TYPO3 version at http://www.hisparc.nl/typo3/ -> 4.5

Then check the requirements for the new TYPO3 version to see if an update of any package is required.


Upgrade from 3.2.3 to 3.5.0
---------------------------

Nagios can be updated to 3.5.0 on CentOS, from the rpm/yum `CentALT <http://pkgs.org/centos-5-rhel-5/centalt-i386/nagios-3.5.0-1.el5.i386.rpm.html>`_ repo.

.. code-block:: sh

    $ cd /tmp
    $ wget http://centos.alt.ru/repository/centos/5/i386/nagios-3.5.0-1.el5.i386.rpm
    $ rpm -i --test nagios-3.5.0-1.el5.i386.rpm 
    warning: nagios-3.5.0-1.el5.i386.rpm: Header V3 DSA signature: NOKEY, key ID e9bc4ae1
	file /etc/httpd/conf.d/nagios.conf from install of nagios-3.5.0-1.el5.i386 conflicts with file from package nagios-3.2.3-3.el5.rf.i386
	file /etc/nagios/cgi.cfg from install of nagios-3.5.0-1.el5.i386 conflicts with file from package nagios-3.2.3-3.el5.rf.i386
	file /etc/nagios/nagios.cfg from install of nagios-3.5.0-1.el5.i386 conflicts with file from package nagios-3.2.3-3.el5.rf.i386
	file /etc/nagios/objects/commands.cfg from install of nagios-3.5.0-1.el5.i386 conflicts with file from package nagios-3.2.3-3.el5.rf.i386
	file /etc/nagios/objects/timeperiods.cfg from install of nagios-3.5.0-1.el5.i386 conflicts with file from package nagios-3.2.3-3.el5.rf.i386
	file /etc/rc.d/init.d/nagios from install of nagios-3.5.0-1.el5.i386 conflicts with file from package nagios-3.2.3-3.el5.rf.i386
	file /usr/bin/nagiostats from install of nagios-3.5.0-1.el5.i386 conflicts with file from package nagios-3.2.3-3.el5.rf.i386

Check the current installed versions and available updates.

.. code-block:: sh

    $ yum list all | grep nagios
    nagios.i386                 3.2.3-3.el5.rf              installed
    nagios.i386                 3.5.0-1.el5                 CentALT  
    nagios-devel.i386           3.5.0-1.el5                 CentALT
    nagios-plugins.i386         1.4.15-2.el5.rf             installed
    nagios-plugins.i386         1.4.16-1.el5.rf             rpmforge 
    nagios-plugins-nrpe.i386    2.12-1.el5.rf               installed
    nagios-plugins-nrpe.i386    2.14-1.el5.rf               rpmforge
    nagios-nsca.i386            2.7.2-2.el5.rf              installed
    nagios-nsca.i386            2.7.2-4.el5.rf              rpmforge 
    nagios-nsca-client.i386     2.7.2-2.el5.rf              installed
    nagios-nsca-client.i386     2.7.2-4.el5.rf              rpmforge 


Install update
^^^^^^^^^^^^^^

Stop Nagios, NSCA and Apache.
(Nagios was running in a screen, not yet as service, so stop it there.)

.. code-block:: sh

    $ screen -r [..]
    ^c
    $ exit
    $ sudo /sbin/service nsca stop
    $ sudo /sbin/service httpd stop

    $ yum update nagios nagios-plugins nagios-plugins-nrpe nagios-nsca nagios-nsca-client
    > y
    Updating : nagios
     warning: /etc/httpd/conf.d/nagios.conf created as /etc/httpd/conf.d/nagios.conf.rpmnew
     warning: /etc/nagios/cgi.cfg created as /etc/nagios/cgi.cfg.rpmnew
     warning: /etc/nagios/nagios.cfg created as /etc/nagios/nagios.cfg.rpmnew
     warning: /etc/nagios/objects/commands.cfg created as /etc/nagios/objects/commands.cfg.rpmnew
     warning: /etc/rc.d/init.d/nagios saved as /etc/rc.d/init.d/nagios.rpmsave
    Updated:
     nagios.i386 0:3.5.0-1.el5
     nagios-plugins.i386 0:1.4.16-1.el5.rf
     nagios-plugins-nrpe.i386 0:2.14-1.el5.rf
     nagios-nsca.i386 0:2.7.2-4.el5.rf
     nagios-nsca-client.i386 0:2.7.2-4.el5.rf


Update config files
^^^^^^^^^^^^^^^^^^^

Using the \*.rpmnew files as guides.

/etc/httpd/conf.d/nagios.conf::

    - /usr/share/nagios
    + /usr/share/nagios/html
    - /usr/lib/nagios/cgi/
    + /usr/lib/nagios/cgi-bin/

/etc/nagios/cgi.cfg::

    - physical_html_path=/usr/share/nagios/
    + physical_html_path=/usr/share/nagios/html
    + result_limit=0

/etc/nagios/nagios.cfg::

    - log_file=/var/nagios/nagios.log
    + log_file=/var/log/nagios/nagios.log
    - object_cache_file=/var/nagios/objects.cache
    + object_cache_file=/var/log/nagios/objects.cache
    - precached_object_file=/var/nagios/objects.precache
    + precached_object_file=/var/log/nagios/objects.precache

    - resource_file=/etc/nagios/resource.cfg
    + resource_file=/etc/nagios/private/resource.cfg

    - status_file=/var/nagios/status.dat
    + status_file=/var/log/nagios/status.dat

    ? command_file=/var/spool/nagios/cmd/nagios.cmd

    - lock_file=/var/run/nagios.pid
    + lock_file=/tmp/nagios.pid

    - temp_file=/var/nagios/nagios.tmp
    + temp_file=/var/log/nagios/nagios.tmp  
  
    ? log_archive_path=/var/log/nagios/archives

    - state_retention_file=/var/nagios/retention.dat
    + state_retention_file=/var/log/nagios/retention.dat
    + service_check_timeout_state=c

    - p1_file=/usr/bin/p1.pl
    + p1_file=/usr/sbin/p1.pl

    - admin_email=davidf@nikhef.nl
    - admin_pager=davidf@nikhef.nl
    + admin_email=adelaat@nikhef.nl
    + admin_pager=adelaat@nikhef.nl


/etc/nagios/objects/commands.cfg::

    - /var/nagios/host-perfdata.out
    - /var/nagios/service-perfdata.out
    + /var/log/nagios/host-perfdata.out
    + /var/log/nagios/service-perfdata.out

/etc/nagios/nsca.cfg::

    - command_file=/var/nagios/rw/nagios.cmd
    + command_file=/var/spool/nagios/cmd/nagios.cmd
    - alternate_dump_file=/var/nagios/rw/nsca.dump
    + alternate_dump_file=/var/log/nagios/rw/nsca.dump


Verify config
^^^^^^^^^^^^^

.. code-block:: sh

    $ nagios -v /etc/nagios/nagios.cfg


Fix Nagios Daemon
^^^^^^^^^^^^^^^^^

Before the update Nagios gave these errors when trying to start as service:

.. code-block:: sh

    $ service nagios start
    log:
    [1365710071] Failed to obtain lock on file /var/run/nagios.pid: Permission denied
    [1365710071] Bailing out due to errors encountered while attempting to daemonize... (PID=15845)

From: `Failed to obtain lock on file /var/run/nagios.pid <http://h3x.no/2010/06/30/failed-to-obtain-lock-on-file-varrunnagios-pid-permission-denied>`_

Changing the value of ``lock_file`` in ``/etc/nagios/nagios.cfg`` from ``/var/run/nagios.pid`` to ``/tmp/nagios.pid`` allowed the PID to be written and the init script to succeed.

Change lock_file location to a place with read+write permissions.


Start services
^^^^^^^^^^^^^^

.. code-block:: sh

    $ sudo /sbin/service httpd start
    $ sudo /sbin/service nagios start
    $ sudo /sbin/service nsca start


Upgrade Apache to latest 2.2.x
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The latest Apache is in the CentALT repository

.. code-block:: sh

    $ yum list httpd
    Installed Packages
    httpd.i386                  2.2.3-53.el5.centos.3       installed
    Available Packages
    httpd.i386                  2.2.24-1.el5                CentALT


Stop services

.. code-block:: sh

    $ sudo /sbin/service httpd stop
    $ sudo /sbin/service nagios stop
    $ sudo /sbin/service nsca stop

Update Apache

.. code-block:: sh

    $ yum update httpd
    Dependencies Resolved

    ======================================================================
     Package                 Arch    Version          Repository     Size
    ======================================================================
    Updating:
     httpd                   i386    2.2.24-1.el5     CentALT       1.3 M
    Installing for dependencies:
     apr-util-ldap           i386    1.4.1-1.el5      CentALT        14 k
     httpd-tools             i386    2.2.24-1.el5     CentALT        68 k
    Updating for dependencies:
     apr-util                i386    1.4.1-1.el5      CentALT        82 k

    Transaction Summary
    ======================================================================
    Install       2 Package(s)
    Upgrade       2 Package(s)

    > y

    warning: /etc/httpd/conf/httpd.conf created as /etc/httpd/conf/httpd.conf.rpmnew

    Dependency Installed:
     apr-util-ldap.i386 0:1.4.1-1.el5
     httpd-tools.i386 0:2.2.24-1.el5                       

    Updated:
      httpd.i386 0:2.2.24-1.el5                                                                                     

    Dependency Updated:
      apr-util.i386 0:1.4.1-1.el5       

Todo: update httpd.conf with new options from httpd.conf.rpmnew

Start services

.. code-block:: sh

    $ sudo /sbin/service httpd start
    $ sudo /sbin/service nagios start
    $ sudo /sbin/service nsca start
