Installation of Tietar
======================

.. note:: New nagios.cfg config and postfix config, template.cfg,
   shorewall rules

This server was originally installed by Tristan in May 2008.  Only
recently did David start making changes to the system.  Later changes are
documented here.  Hopefully, they will be expanded to include a
description of the complete system.

Due to a partial disk crash February 18th, 2010, we reinstalled the
system.  Due to lack of time, a lot of the original configuration was
retrieved from backups without analyzing the design.

Installation
------------

Adding user davidf:

.. code-block:: sh

    (root)$ adduser davidf
    (root)$ passwd davidf

Granting davidf rights to manage software and services:

.. code-block:: sh

    (root)$ visudo

and adding::

    davidf  ALL = SOFTWARE, SERVICES


Adding the hisparc group
------------------------

We've added the hisparc group to the system and made a few users part of it:

.. code-block:: sh

    (root)$ groupadd hisparc
    (root)$ usermod -G hisparc davidf


Preparing for source install
----------------------------

Issue:

.. code-block:: sh

    (root)$ cd /usr/local/src/
    (root)$ mkdir hisparc
    (root)$ chown davidf.hisparc hisparc/
    $ chmod g+sw hisparc/

In /etc/ld.so.conf.d new file usrlocal.conf, to let ldconfig find
libraries of locally installed software::

    /usr/local/lib

Then, install a compiler:

.. code-block:: sh

    $ sudo yum install gcc


Setting up RPMForge
-------------------

RPMForge provides extra packages for CentOS, including Nagios and more
recent versions of the SSL libraries.  To enable it:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ wget http://packages.sw.be/rpmforge-release/rpmforge-release-0.5.1-1.el5.rf.i386.rpm
    $ sudo rpm --import http://dag.wieers.com/rpm/packages/RPM-GPG-KEY.dag.txt
    $ rpm -K rpmforge-release-0.5.1-1.el5.rf.*.rpm
    $ sudo rpm -i rpmforge-release-0.5.1-1.el5.rf.*.rpm

Check succesful installation with and update packages::

    $ sudo yum check-update
    $ sudo yum update


Python
------

Prerequisites for standard libraries:

.. code-block:: sh

    $ sudo yum install zlib-devel
    $ sudo yum install bzip2-devel

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


Web server
----------

Install apache:

.. code-block:: sh

    $ sudo yum install httpd

    ================================================================================
     Package              Arch      Version                      Repository    Size
    ================================================================================
    Installing:
     httpd                i386      2.2.3-31.el5.centos.2        updates      1.2 M
    Installing for dependencies:
     apr                  i386      1.2.7-11.el5_3.1             base         123 k
     apr-util             i386      1.2.7-7.el5_3.2              base          76 k
     postgresql-libs      i386      8.1.18-2.el5_4.1             updates      196 k

Enabling httpd on startup::

    $ sudo /sbin/chkconfig --levels 35 httpd on

Starting httpd now::

    $ sudo /sbin/service httpd start


OpenVPN
-------

Install OpenVPN from source, as we require version 2.1.1, which has no
official RPM:

.. code-block:: sh

    $ sudo yum install lzo2-devel
    $ sudo yum install openssl-devel
    $ wget http://openvpn.net/release/openvpn-2.1.1.tar.gz
    $ tar xvzf openvpn-2.1.1.tar.gz 
    $ cd openvpn-2.1.1
    $ ./configure
    $ make
    (root)$ make install

Blindly copy old configuration, but changed one directory name:

.. code-block:: sh

    (root)$ cp -r /mnt/oldroot/etc/openvpn/* .
    $ cd /etc/openvpn
    (root)$ mv easy-rsa easy_rsa

To add OpenVPN as a service and start it:

.. code-block:: sh

    $ cd /usr/local/src/hisparc/openvpn-2.1.1/sample-scripts/
    (root)$ cp openvpn.init /etc/init.d/openvpn
    $ sudo /sbin/chkconfig --add openvpn
    $ sudo /sbin/service openvpn start


Dnsmasq
-------

Dnsmasq handles our DNS requirements.  On this system, it was already
installed.  Edited configuration, with the following resulting diff::

    --- dnsmasq.conf.orig   2010-02-22 10:59:01.000000000 +0100
    +++ dnsmasq.conf        2010-02-25 13:43:19.000000000 +0100
    @@ -13,7 +13,7 @@
     # Never forward plain names (without a dot or domain part)
     #domain-needed
     # Never forward addresses in the non-routed address spaces.
    -#bogus-priv
    +bogus-priv
     
     
     # Uncomment this to filter useless windows-originated DNS requests
    @@ -26,7 +26,7 @@
     
     # Change this line if you want dns to get its upstream servers from
     # somewhere other that /etc/resolv.conf
    -#resolv-file=
    +resolv-file=/etc/resolv.conf-nikhef
     
     # By  default,  dnsmasq  will  send queries to any of the upstream
     # servers it knows about and tries to favour servers to are  known
    @@ -55,6 +55,7 @@
     # Add local-only domains here, queries in these domains are answered
     # from /etc/hosts or DHCP only.
     #local=/localnet/
    +local=/his/
     
     # Add domains which you want to force to an IP address here.
     # The example below send any host in doubleclick.net to a local
    @@ -85,6 +86,7 @@
     #interface=
     # Or you can specify which interface _not_ to listen on
     #except-interface=
    +except-interface=eth0
     # Or which to listen on by address (remember to include 127.0.0.1 if
     # you use this.)
     #listen-address=
    @@ -108,10 +110,11 @@
     # or if you want it to read another file, as well as /etc/hosts, use
     # this.
     #addn-hosts=/etc/banner_add_hosts
    +addn-hosts=/etc/hosts-hisparc
     
     # Set this (and domain: see below) if you want to have a domain
     # automatically added to simple names in a hosts-file.
    -#expand-hosts
    +expand-hosts
     
     # Set the domain for dnsmasq. this is optional, but if it is set, it
     # does the following things.
    @@ -121,6 +124,7 @@
     #    domain of all systems configured by DHCP
     # 3) Provides the domain part for "expand-hosts"
     #domain=thekelleys.org.uk
    +domain=his
     
     # Set a different domain for a particular subnet
     #domain=wireless.thekelleys.org.uk,192.168.2.0/24

Copy /etc/resolv.conf to /etc/resolv.conf-nikhef and edit /etc/resolv.conf
to contain::

    search nikhef.nl his
    nameserver 127.0.0.1

Enabling dnsmasq on startup and start it for the first time:

.. code-block:: sh

    $ sudo /sbin/chkconfig --level 35 dnsmasq on
    $ sudo /sbin/service dnsmasq start


Nagios
------

Install nagios from RPMForge:

.. code-block:: sh

    $ sudo yum install nagios nagios-plugins nagios-plugins-nrpe nagios-nsca
    $ sudo /sbin/chkconfig --level 35 nsca on

Edited several configuration files::

    --- nagios.conf.orig    2010-02-22 13:50:14.000000000 +0100
    +++ /etc/httpd/conf.d/nagios.conf 2010-02-22 13:50:31.000000000 +0100
    @@ -17,10 +17,10 @@
     #  Order deny,allow
     #  Deny from all
     #  Allow from 127.0.0.1
    -   AuthName "Nagios Access"
    -   AuthType Basic
    -   AuthUserFile /etc/nagios/htpasswd.users
    -   Require valid-user
    +#   AuthName "Nagios Access"
    +#   AuthType Basic
    +#   AuthUserFile /etc/nagios/htpasswd.users
    +#   Require valid-user
     </Directory>
     
     Alias /nagios "/usr/share/nagios"
    @@ -34,9 +34,9 @@
     #  Order deny,allow
     #  Deny from all
     #  Allow from 127.0.0.1
    -   AuthName "Nagios Access"
    -   AuthType Basic
    -   AuthUserFile /etc/nagios/htpasswd.users
    -   Require valid-user
    +#   AuthName "Nagios Access"
    +#   AuthType Basic
    +#   AuthUserFile /etc/nagios/htpasswd.users
    +#   Require valid-user
     </Directory>


    --- cgi.cfg.orig        2010-02-22 13:41:05.000000000 +0100
    +++ /etc/nagios/cgi.cfg 2010-02-26 11:44:01.000000000 +0100
    @@ -105,6 +105,7 @@
     # server will inherit all rights you assign to this user!
      
     #default_user_name=guest
    +default_user_name=nagiosadmin
     
     
     
    @@ -272,7 +273,7 @@
     # This option allows you to specify the refresh rate in seconds
     # of various CGIs (status, statusmap, extinfo, and outages).  
     
    -refresh_rate=90
    +refresh_rate=30


    --- nagios.cfg.orig     2010-02-22 13:37:45.000000000 +0100
    +++ /etc/nagios/nagios.cfg  2010-02-22 15:05:03.000000000 +0100
    @@ -33,7 +33,7 @@
     cfg_file=/etc/nagios/objects/templates.cfg
     
     # Definitions for monitoring the local (Linux) host
    -cfg_file=/etc/nagios/objects/localhost.cfg
    +#cfg_file=/etc/nagios/objects/localhost.cfg
     
     # Definitions for monitoring a Windows machine
     #cfg_file=/etc/nagios/objects/windows.cfg
    @@ -44,6 +44,9 @@
     # Definitions for monitoring a network printer
     #cfg_file=/etc/nagios/objects/printer.cfg
     
    +# Definitions for HiSPARC
    +cfg_file=/etc/nagios/objects/hisparc.cfg
    +
     
     # You can also tell Nagios to process all config files (with a .cfg
     # extension) in a particular directory by using the cfg_dir


    --- nsca.cfg.orig       2010-02-22 15:38:01.000000000 +0100
    +++ /etc/nagios/nsca.cfg    2010-02-22 15:38:06.000000000 +0100
    @@ -187,5 +187,5 @@
     #      26 = SAFER+
     #
     
    -decryption_method=1
    +decryption_method=0


    --- commands.cfg.orig   2010-02-22 15:06:44.000000000 +0100
    +++ /etc/nagios/objects/commands.cfg        2010-02-22 15:18:59.000000000 +0100
    @@ -237,4 +237,19 @@
            command_line    /usr/bin/printf "%b" "$LASTSERVICECHECK$\t$HOSTNAME$\t$SERVICEDESC$\t$SERVICESTATE$\t$SERVICEATTEMPT$\t$SERVICESTATETYPE$\t$SERVICEEXECUTIONTIME$\t$SERVICELATENCY$\t$SERVICEOUTPUT$\t$SERVICEPERFDATA$\n" >> /var/nagios/service-perfdata.out
            }
     
    +# NRPE!
     
    +define command{
    +        command_name check_nrpe
    +        command_line $USER1$/check_nrpe -t 30 -H $HOSTADDRESS$ -c $ARG1$ -a $ARG2$ $ARG3$
    +}
    +
    +define command{
    +        command_name check_mysql
    +        command_line $USER1$/check_mysql -H $HOSTADDRESS$ -u $ARG1$ -p $ARG2$
    +}
    +
    +define command{
    +        command_name check_dummy
    +        command_line $USER1$/check_dummy $ARG1$ $ARG2$
    +}

Reload apache configuration and start nagios:

.. code-block:: sh

    $ sudo /sbin/service httpd reload
    $ sudo /sbin/service nagios start
    $ sudo /sbin/service nsca start


Version control
---------------

Install git from source:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ wget https://git-core.googlecode.com/files/git-1.8.4.3.tar.gz
    $ tar xvzf git-1.8.4.3.tar.gz
    $ cd git-1.8.4.3.tar.gz
	$ make prefix=/usr/local all
	(root)$ sudo make prefix=/usr/local install


Paramiko
^^^^^^^^

Paramiko supports ssh2 for python, which is needed to do a checkout of our
application's sources over sftp.  Install using easy_install:

.. code-block:: sh

    (root)$ easy_install paramiko

This will automatically download, compile and install dependencies
(pycrypto).


Setting up the HiSPARC public database scripts
----------------------------------------------

First, do a checkout of the public database sources:

.. code-block:: sh

    $ cd /usr/local/src/hisparc
    $ git clone https://github.com/HiSPARC/publicdb.git publicdb

Symlink the vpn server example scripts into /usr/local/bin:

.. code-block:: sh

    (root)$ ln -s /usr/local/src/hisparc/publicdb/examples/create_admin_keys.sh .
    (root)$ ln -s /usr/local/src/hisparc/publicdb/examples/create_keys.sh .
    (root)$ ln -s /usr/local/src/hisparc/publicdb/examples/vpn-cron.py hisparc-nagios
    (root)$  ln -s /usr/local/src/hisparc/publicdb/examples/vpn-xmlrpc-server.py hisparcvpnd

And set execute permissions:

.. code-block:: sh

    $ cd /usr/local/src/hisparc/publicdb/examples
    $ chmod +x vpn-cron.py 
    $ chmod +x vpn-xmlrpc-server.py 

Change some paths and host information, resulting in the following diff::

    === modified file 'examples/vpn-cron.py'
    --- examples/vpn-cron.py        2010-01-15 21:36:15 +0000
    +++ examples/vpn-cron.py        2010-02-22 11:32:43 +0000
    @@ -1,4 +1,4 @@
    -#!/usr/bin/python
    +#!/usr/local/bin/python
     """ Reload nagios if necessary
     
         This script checks for the existence of the nagios restart flag,

    === modified file 'examples/vpn-xmlrpc-server.py'
    --- examples/vpn-xmlrpc-server.py       2010-01-15 14:31:24 +0000
    +++ examples/vpn-xmlrpc-server.py       2010-02-22 11:35:27 +0000
    @@ -1,4 +1,4 @@
    -#!/usr/bin/python
    +#!/usr/local/bin/python
     """ Simple XML-RPC Server to run on the VPN server
     
         This daemon should be run on HiSPARC's VPN server.  It will handle the
    @@ -17,21 +17,22 @@
     import os
     import base64
     
    -OPENVPN_DIR = '/home/david/tmp/openvpn'
    -HOSTS_FILE = '/tmp/hosts-hisparc'
    +OPENVPN_DIR = '/etc/openvpn'
    +HOSTS_FILE = '/etc/hosts-hisparc'
     FLAG = '/tmp/flag_nagios_reload'
     
     def create_key(host, type, ip):
         """create keys for a host and set up openvpn"""
     
         if type == 'client':
    -        subprocess.check_call(['./create_keys.sh', OPENVPN_DIR, host])
    +        subprocess.check_call(['/usr/local/bin/create_keys.sh', OPENVPN_DIR,
    +                                host])
             with open(os.path.join(OPENVPN_DIR, 'ccd', host), 'w') as file:
                 file.write('ifconfig-push %s 255.255.254.0 194.171.82.1\n' %
                            ip)
         elif type == 'admin':
    -        subprocess.check_call(['./create_admin_keys.sh', OPENVPN_DIR,
    -                               host])
    +        subprocess.check_call(['/usr/local/bin/create_admin_keys.sh',
    +                              OPENVPN_DIR, host])
         else:
             raise Exception('Unknown type %s' % type)
     
    @@ -89,7 +90,7 @@
             rpc_paths = ('/RPC2',)
     
         # Create server
    -    server = SimpleXMLRPCServer(("localhost", 8001),
    +    server = SimpleXMLRPCServer(("tietar.nikhef.nl", 8001),
                                     requestHandler=RequestHandler)
         server.register_introspection_functions()

To set up the cron job for reloading nagios config, execute:

.. code-block:: sh

    (root)$ crontab -e

and add::

    # Run nagios reload check every minute
    * * * * * /usr/local/bin/hisparc-nagios


Shoreline Firewall (Shorewall)
------------------------------

Get an RPM from:

.. code-block:: sh

    $ wget http://slovakia.shorewall.net/pub/shorewall/4.4/shorewall-4.4.7/shorewall-4.4.7-5.noarch.rpm
    $ sudo rpm -i shorewall-4.4.7-5.noarch.rpm 

There is a lot of configuration to change.  After thoroughly checking the
existing configuration, I decided that it was not very clean.  Some
relevant options were missing and things were not documented very well.

For the new configuration, we start with our zones file::

    --- zones.orig  2010-02-25 11:22:18.000000000 +0100
    +++ zones       2010-02-25 11:23:52.000000000 +0100
    @@ -10,3 +10,6 @@
     #ZONE  TYPE            OPTIONS         IN                      OUT
     #                                      OPTIONS                 OPTIONS
     fw     firewall
    +net    ipv4
    +det    ipv4
    +adm    ipv4

with the matching interfaces file::

    --- interfaces.orig     2010-02-25 11:51:46.000000000 +0100
    +++ interfaces  2010-02-25 12:05:52.000000000 +0100
    @@ -8,3 +8,6 @@
     #
     ###############################################################################
     #ZONE  INTERFACE       BROADCAST       OPTIONS
    +net    eth0            detect          logmartians,nosmurfs,routefilter,tcpflags
    +det    tun1            detect          logmartians,nosmurfs,routefilter,tcpflags
    +adm    tun0            detect          logmartians,nosmurfs,routefilter,tcpflags

First, we'll define the policy::

    --- policy.orig 2010-02-25 11:29:47.000000000 +0100
    +++ policy      2010-02-25 11:46:41.000000000 +0100
    @@ -9,3 +9,22 @@
     ###############################################################################
     #SOURCE        DEST    POLICY          LOG     LIMIT:          CONNLIMIT:
     #                              LEVEL   BURST           MASK
    +
    +# The firewall may connect to the internet
    +$FW    net     ACCEPT
    +
    +# The internet should not be aware of any services running on the
    +# firewall, except for a few exceptions (see rules)
    +net    all     DROP            info
    +
    +# HiSPARC detector pc's should never route traffic over their VPN
    +# interfaces, except for a few exceptions (see rules)
    +det    net     DROP            err
    +det    adm     DROP            err
    +
    +# HiSPARC admins should never route internet traffic over their VPN
    +# interfaces
    +adm    net     DROP            err
    +
    +# All other connections: reject
    +all    all     REJECT          info

To easily enable the VPN traffic, without having to add various exception rules, we can define the VPN tunnels in the tunnels file::

    --- tunnels.orig        2010-02-25 13:26:53.000000000 +0100
    +++ tunnels     2010-02-25 13:29:56.000000000 +0100
    @@ -9,3 +9,9 @@
     ###############################################################################
     #TYPE                  ZONE    GATEWAY         GATEWAY
     #                                              ZONE
    +
    +# Admin VPN
    +openvpnserver          net     0.0.0.0/0
    +
    +# Detector VPN
    +openvpnserver:tcp:443  net     0.0.0.0/0

The rest of the traffic has to be enabled by adding exceptions to the rules file::

    --- rules.orig  2010-02-25 11:50:52.000000000 +0100
    +++ rules       2010-02-25 14:06:13.000000000 +0100
    @@ -12,3 +12,39 @@
     #SECTION ESTABLISHED
     #SECTION RELATED
     SECTION NEW
    +
    +# Always accept SSH to tietar
    +SSH(ACCEPT)    all             $FW
    +# Accept SSH from detector vpn to admin vpn
    +SSH(ACCEPT)    det             adm
    +
    +# Accept ping to firewall and icmp from firewall
    +Ping(ACCEPT)   all             $FW
    +ACCEPT         $FW             all             icmp
    +# Accept ping from admin vpn to detector vpn
    +Ping(ACCEPT)   adm             det
    +
    +#
    +# Services running on tietar
    +#
    +# DNS
    +DNS(ACCEPT)    det             $FW
    +DNS(ACCEPT)    adm             $FW
    +# Web
    +Web(ACCEPT)    net             $FW
    +# vpn xml-rpc server (allowed from pique)
    +ACCEPT         net:192.16.185.167      $FW             tcp     8001
    +
    +#
    +# Nagios traffic
    +#
    +# NRPE, NSClient running on detector pc's
    +ACCEPT         $FW             det     tcp     5666,12489
    +# NSCA running on detector pc's
    +ACCEPT         det             $FW     tcp     5667
    +
    +#
    +# Admin access to detector pc's
    +#
    +# VNC
    +ACCEPT         adm             det     tcp     5900

Our firewall is now set up.  To keep the server accessible when the firewall is stopped, starting or stopping, we can edit the routestopped file::

    --- routestopped.orig   2010-02-25 12:39:00.000000000 +0100
    +++ routestopped        2010-02-25 12:39:59.000000000 +0100
    @@ -12,3 +12,4 @@
     ###############################################################################
     #INTERFACE     HOST(S)                 OPTIONS         PROTO   DEST    SOURCE
     #                                                              PORT(S) PORT(S)
    +eth0           -                       -               tcp     ssh

where we've only enabled SSH access.  The only thing remaining is enabling
our firewall::

    --- shorewall.conf.orig 2010-02-25 12:33:32.000000000 +0100
    +++ shorewall.conf      2010-02-25 14:33:41.000000000 +0100
    @@ -18,7 +18,7 @@
     #                     S T A R T U P   E N A B L E D
     ###############################################################################
     
    -STARTUP_ENABLED=No
    +STARTUP_ENABLED=Yes
     
     ###############################################################################
     #                            V E R B O S I T Y

Starting our firewall is accomplished with:

.. code-block:: sh

    $ sudo /sbin/service shorewall start


(Maybe) not relevant
--------------------

Installed screen
Installed ntp
