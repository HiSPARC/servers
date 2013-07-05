Upgrading of TYPO3
==================

This describes how to update `TYPO3 <http://www.typo3.org>`_. General
Upgrade instructions are available on `TYPO3 Upgrade
<http://wiki.typo3.org/Upgrade>`_. This guide is about applying these
updates on neckar (:doc:`neckar`).


Current versions
----------------

First check the versions of the OS, MySQL, Apache, PHP and TYPO3.

.. code-block:: sh

    $ cat /etc/redhat-release
    CentOS release 5.9 (Final)
    $ mysql --version
    mysql  Ver 14.12 Distrib 5.0.95
    $ httpd -v
    Server version: Apache/2.2.3
    $ php -v
    PHP 5.3.19
    $ yum list installed | grep php
    php.x86_64            5.3.19-1.w5
    php-cli.x86_64        5.3.19-1.w5
    php-common.x86_64     5.3.19-1.w5
    php-gd.x86_64         5.3.19-1.w5
    php-mysql.x86_64      5.3.19-1.w5
    php-pdo.x86_64        5.3.19-1.w5

Check TYPO3 version at http://www.hisparc.nl/typo3/ -> 4.5

Then check the requirements for the new TYPO3 version to see if an
update of any package is required.


Source code location
--------------------

Here we download the source code of the update. replace [version number]
with the version number.

.. code-block:: sh

    $ cd /usr/local/src
    $ wget http://garr.dl.sourceforge.net/project/typo3/TYPO3%20Source%20and%20Dummy/TYPO3%20[version number]/typo3_src-[version number].tar.gz
    $ tar xfz typo3_src-[version number].tar.gz
    $ chown www.www *


Upgrading TYPO3 version
-----------------------


Check requirements and deprecations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Check the `TYPO3 Upgrade <http://wiki.typo3.org/Upgrade>`_ page for any
new requirements and deprecations.


Make MySQL backup
^^^^^^^^^^^^^^^^^

Make a backup of the TYPO3 MySQL database. This dumps the current
hisparc\_t3 database in the hisparc\_t3.sql file.

.. code-block:: sh

    $ mysqldump -u hisparc -p hisparc_t3 > hisparc_t3.sql
    > [enter MySQL password]

To restore the MySQL database from a backup in case of corruption or an error.

.. note:: The .sql file contains ``DROP TABLE`` commands to overwrite the existing tables.

.. code-block:: sh

    $ mysql -u hisparc -p hisparc_t3 < hisparc_t3.sql
    > [enter MySQL password]


Use new source files
^^^^^^^^^^^^^^^^^^^^

Link to new source files, correct permissions and enable TYPO3 install
tool.

.. code-block:: sh

    $ cd /usr/local/www/web
    $ ln -f -s /usr/local/src/typo3_src-[version number]/* .
    $ chown -h www.www *


Install the new version
^^^^^^^^^^^^^^^^^^^^^^^

Enable the Install Tool

.. code-block:: sh

    $ touch typo3conf/ENABLE_INSTALL_TOOL

Now run the Install Tool to migrate to the new version. The database
needs to be updated, several times. Then the Wizard will easily guide
you through the changes that need to be made.

1. Enter the install tool.
    - http://www.hisparc.nl/typo3/install/
    - Enter the Password.
2. Analyse and update database:
    - Go to section "Database Analyzer".
    - Click "Update required tables".
    - Click "COMPARE" and "IMPORT" and apply the proposed changes.
3. Go through the Upgrade Wizard:
    - Go to section "Upgrade Wizard".
    - Set the compatibility version.
    - Go through the other proposed changes.
4. Remove temp\_CACHED files:
    - Go to section "Edit files in typo3conf/".
    - Choose the option 'Delete all temp_CACHED* files'.
5. Update DB Reference index
    - In the Backend click on "DB Check" under "Admin Tools".
    - Select "Manage Reference index" from the drop down list.
    - Run "Check reference index", if there are changes to be made, click "update reference index".


Upgrade the extensions
^^^^^^^^^^^^^^^^^^^^^^

Update TYPO3 Extensions in Ext Manager -> Check for extension updates;
Be careful not to update extensions to the very latest version, check
version compatibility. Remove any unused extensions. Use the ``Useful
informations in the reports module`` to check the usage of the extensions


Upgrade from 4.2.8 (to 4.3.14) to 4.4.15
----------------------------------------

Follow the `Upgrading TYPO3 version`_ instructions above.

TYPO3 4.3+ requires PHP 5.2.0 or newer with the following extensions:
filter, *GD2*, JSON, mysql, pcre, session, SPL, standard, xml


Upgrade PHP from 5.1.6 to 5.3.19
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CentOS 5.x comes with PHP 5.1.x, but 5.3 is required, there is a php53
package, but a package named php can also be found, which is also more
up to date.

http://www.webtatic.com/packages/php53/

.. code-block:: sh

    $ rpm -Uvh http://repo.webtatic.com/yum/centos/5/latest.rpm
    $ yum --enablerepo=webtatic update php
     Package                  Arch     Version       Repository   Size
    ===================================================================
    Updating:
     php                      x86_64   5.3.19-1.w5   webtatic    1.4 M
    Updating for dependencies:
     php-cli                  x86_64   5.3.19-1.w5   webtatic    2.6 M
     php-common               x86_64   5.3.19-1.w5   webtatic    661 k
     php-mysql                x86_64   5.3.19-1.w5   webtatic     91 k
     php-pdo                  x86_64   5.3.19-1.w5   webtatic     66 k
    Upgrade       5 Package(s)
    Total download size: 4.8 M
    Is this ok [y/N]:
    $ y
    warning: rpmts_HdrFromFdno: Header V3 DSA signature: NOKEY, key ID cf4c4ff9
    Importing GPG key 0xCF4C4FF9 "Andy Thompson <andy@webtatic.com>" from /etc/pki/rpm-gpg/RPM-GPG-KEY-webtatic-andy
    Is this ok [y/N]:
    $ y
    Updated:
      php.x86_64 0:5.3.19-1.w5                                                                                                                                
    Dependency Updated:
      php-cli.x86_64 0:5.3.19-1.w5     php-common.x86_64 0:5.3.19-1.w5   
      php-mysql.x86_64 0:5.3.19-1.w5   php-pdo.x86_64 0:5.3.19-1.w5
    $ php -v
    PHP 5.3.19 (cli) (built: Nov 25 2012 13:46:54)
    $ /sbin/service httpd reload

Other possibility: `Update CentOS 5 PHP 5.1 to PHP 5.3
<http://www.andresmontalban.com/update-centos-5-php-5-1-to-php-5-3/>`_


Install missing PHP module
^^^^^^^^^^^^^^^^^^^^^^^^^^

Check installed modules using a simple php page with:

.. code-block:: php

    <?php phpinfo() ?>

It appears that GD is not yet installed.

.. code-block:: sh

    $ yum --enablerepo=webtatic install php-gd
     Package      Arch    Version         Repository     Size
    ==========================================================
    Installing:
     php-gd       x86_64  5.3.19-1.w5     webtatic      108 k
    Install       1 Package(s)
    Total download size: 108 k
    Is this ok [y/N]:
    $ y
    Installed:
      php-gd.x86_64 0:5.3.19-1.w5                                                                                                                             
    $ /sbin/service httpd reload


Add gzipping to .htaccess
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: sh

    $ vim .htaccess
    <FilesMatch "\.js\.gzip$">
      AddType "text/javascript" .gzip
    </FilesMatch>
    <FilesMatch "\.css\.gzip$">
      AddType "text/css" .gzip
    </FilesMatch>
    AddEncoding gzip .gzip


Deprecation error GPvar
^^^^^^^^^^^^^^^^^^^^^^^

Deprecation error in the logs::

    Using gpvar in TypoScript getText is deprecated since TYPO3 4.3 - Use gp instead of gpvar.

Look for ``gpvar`` in the Backend, replace ``GPvar`` by ``GP`` and
reload httpd

.. code-block:: sh

    $ /sbin/service httpd reload


Upgrade from 4.4.15 to 4.5.22 LTS
---------------------------------

*This is a Long Term Support version of TYPO3*

Follow the `Upgrading TYPO3 version`_ instructions above.

Update tt_news to 3.1.0, run the included updater.

Modify the file ``typo3conf/ext/tt\_news/ext\_tables.php``::

    -enableConfigValidation = 1
    +enableConfigValidation = 0


Deprecation error, use UTF-8
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This error appeared in the deprecation log located at
``/usr/local/www/web/typo3conf/deprecation_[..].log``::

    This TYPO3 installation is using the $TYPO3_CONF_VARS['SYS']['setDBinit'] property with the following value:

    It looks like UTF-8 is not used for this connection.

    Everything other than UTF-8 is deprecated since TYPO3 4.5.
    The DB, its connection and TYPO3 should be migrated to UTF-8 therefore. Please check your setup.

Update MySQL Tables to UTF-8: `Convert existing database to UTF-8
<http://wiki.typo3.org/UTF-8_support#
Convert_an_already_existing_database_to_UTF-8>`_

Follow 'Possibility 1'

.. code-block:: sh

    $ mysqldump -u hisparc -p --max_allowed_packet=10000000 hisparc_t3 > hisparc_t3_130319.sql
    > [enter password]
    $ cd /usr/local/www/web/fileadmin
    $ wget "http://dcbjht.home.xs4all.nl/typo3/db_utf8_fix.zip"
    $ unzip db_utf8_fix.zip
    $ vim  db_utf8_fix.php

Then go to http://www.hisparc.nl/fileadmin/db_utf8_fix.php , if all OK
-> change TRUE in line 9 to False and reload the page.

Ensure the following config is set::

    $TYPO3_CONF_VARS['SYS']['setDBinit'] = 'SET NAMES utf8;';

The following was already active::

    $TYPO3_CONF_VARS['BE']['forceCharset'] = 'utf-8';

Special characters where not correctly migrated to new encoding. Install
`find_and_replace` extension, using this these occurances were fixed.
The find and replace extension does not fix all occurrences (tt_news).
Also used this to remove unneeded excess from link tags (`-
external-link 'opens in new ...'`)


Upgrade from 4.5.22 to 4.6.15
-----------------------------

Todo.


Upgrade from 4.6.15 to 4.7.7
----------------------------

Todo.


Upgrade from 4.7.7 to 6.0.0
---------------------------

Requires MySQL 5.1.x-5.5.x



Upgrade Apache to latest 2.2.x
------------------------------

The latest Apache is in the CentALT repository, so add that repo:

.. code-block:: sh

    $ cd /etc/yum.repos.d
    $ vim centos.alt.ru.repo
    [CentALT]
    name=CentALT Packages for Enterprise Linux 5 - $basearch
    baseurl=http://centos.alt.ru/repository/centos/5/$basearch/
    enabled=1
    gpgcheck=0

.. code-block:: sh

    $ yum list httpd
    Installed Packages
    httpd.x86_64                  2.2.3-81.el5.centos.3       installed
    Available Packages
    httpd.x86_64                  2.2.24-1.el5                CentALT

Stop httpd service

.. code-block:: sh

    $ sudo /sbin/service httpd stop

Update Apache

.. code-block:: sh

    $ yum update httpd
    Dependencies Resolved

    ======================================================================
     Package                 Arch    Version          Repository     Size
    ======================================================================
    Updating:
     httpd                   x86_64  2.2.24-1.el5     CentALT       1.3 M
    Installing for dependencies:
     apr-util-ldap           x86_64  1.4.1-1.el5      CentALT        14 k
     httpd-tools             x86_64  2.2.24-1.el5     CentALT        67 k
    Updating for dependencies:
     apr-util                x86_64  1.4.1-1.el5      CentALT        80 k

    Transaction Summary
    ======================================================================
    Install       2 Package(s)
    Upgrade       2 Package(s)

    > y

    warning: /etc/httpd/conf/httpd.conf created as /etc/httpd/conf/httpd.conf.rpmnew

    Dependency Installed:
      apr-util-ldap.x86_64 0:1.4.1-1.el5
      httpd-tools.x86_64 0:2.2.24-1.el5                                         

    Updated:
      httpd.x86_64 0:2.2.24-1.el5                                                                                                                            

    Dependency Updated:
      apr-util.x86_64 0:1.4.1-1.el5                                                                                                                          

Todo: update httpd.conf with new options from httpd.conf.rpmnew

.. warning::

    For some reason the user www is no longer recognized, use apache instead,
    Update ``User`` and ``Group`` in ``/etc/httpd/conf/httpd.conf`` to apache

Start services

.. code-block:: sh

    $ sudo /sbin/service httpd start
