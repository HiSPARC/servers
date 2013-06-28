Installation of Neckar
======================

Granting davidf rights to manage software and services:

.. code-block:: sh

    (root)$ visudo

and adding::

    davidf  ALL = SOFTWARE, SERVICES


Web server
----------

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


MySQL Server
------------

The mysql server was pre-installed on this system, but not configured.  To
configure mysql and create the TYPO3 database:

.. code-block:: sh

    $ sudo /sbin/chkconfig --levels 35 mysqld on
    $ sudo /sbin/service mysqld start
    $ mysqladmin -u root password 'secret_password'
    $ mysql -u root -p
    mysql> create database hisparc_t3 default character set 'utf8';
    mysql> grant all on hisparc_t3.* to 'hisparc'@'localhost' identified by 'secret_password';

HiSPARC website
---------------

The HiSPARC website is a typical TYPO3 installation with some added
modules.  This installation is created and provided by `OOiP
<http://www.ooip.nl>`_.  From the TYPO3 website:

    TYPO3 is a free Open Source content management system for enterprise
    purposes on the web and in intranets.  It offers full flexibility and
    extendability while featuring an accomplished set of ready-made
    interfaces, functions and modules.


Prerequisites
^^^^^^^^^^^^^

TYPO3 has some prerequisites, some of which were already installed: PHP
and ImageMagick.  Unfortunately, MySQL support for PHP was not yet
installed.  Do this by issuing:

.. code-block:: sh

    $ sudo yum install php-mysql

It turns out the permissions for the PHP session directory were incorrect.
Correct them as follows:

.. code-block:: sh

    (root)$ chown www.www /var/lib/php/session

To make sure TYPO3 uses loopback connections to itself, update the
/etc/hosts file to contain::

    127.0.0.1       localhost.localdomain localhost neckar.nikhef.nl neckar www.hisparc.nl


Website
-------

To install the HiSPARC website, untar the OOiP-provided directory dump:

.. code-block:: sh

    $ cd /usr/local
    (root)$ mkdir www
    (root)$ chown www.www www
    $ cd www
    (root)$ tar xvzf hisparc-web.tar.gz --strip-components=1
    (root)$ chown -R www.www *
    (root)$ chmod -R a-x *
    (root)$ chmod -R a+X *
    $ mysql -u hisparc -p hisparc_t3 < hisparc_t3.sql

Create the apache config by creating and editing
/etc/httpd/conf.d/hisparc.conf to contain::

    <VirtualHost *:80>
        ServerName www.hisparc.nl
        ServerAlias neckar.nikhef.nl

        DocumentRoot /usr/local/www/web

        <Directory /usr/local/www/web>
            AllowOverride All
            Allow from All

            Options +FollowSymLinks +ExecCGI

        </Directory>

    </VirtualHost>

After that, reload the web server:

.. code-block:: sh

    $ sudo /sbin/service httpd reload

Installation should now be complete.
