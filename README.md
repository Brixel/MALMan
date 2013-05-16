MALMan is a WSGI app

installing MALMan
=================

In this guide we will set up MALMan at /srv/MALMan on Arch Linux, using Apache 
as a webserver and MariaDB for the database. The example commands reflect this. 
The needed commands will varry accondingly if using an other distibution, 
webserver, database server or path.

getting the code
----------------
create a directory to contain the whole program:
    mkdir -p /usr/local/share/webapps
    cd /usr/local/share/webapps
get the source code:
    git clone git://github.com/voidwarranties/MALMan.git

setting up the virtualenv and getting dependencies
--------------------------------------------------
install version 2 of python, pip and python2-virtualenv:
    pacman -S python2 python2-pip python2-virtualenv
create a virtualenv:
    virtualenv-2.7 env
have pip install all required python packages from the python repos:
    env/bin/pip install -r requirements.txt

setting up the database
-----------------------
install, configure and start MariaDB:
    pacman -S mariadb libmariadbclient mariadb-clients
    systemctl start mysqld
    mysql_secure_installation
    systemctl start mysqld
log in to the MySQL server:
    mysql -p -u root
create the MALMan database:
    > CREATE DATABASE MALMan;
create a new user (replace 'password'):
    > CREATE USER MALMan@localhost IDENTIFIED BY 'password';
grant our new user full access to the MALMan database:
    > GRANT ALL PRIVILEGES ON MALMan.* TO MALMan@localhost;
close the MySQL shell:
    > EXIT
import the MALMan database content into the empty MALMan database:
    mysql -u MALMan -p -h localhost MALMan < database.sql

configuration
-------------
copy MALMan.cfg.template to MALMan.cfg 
    cp MALMan/MALMan.cfg{.template,}
adapt the file to fit your setup, so MALMan can acces the MySQL server and SMTP server

running in debug mode
---------------------
You can now run MALMan in dev mode:
    env/bin/python2.7 run.py 
MALMan should be running on 0.0.0.0:5000
you can log in with the following credentials: 
    user: root@example.org
    password: password

running as a WSGI app
---------------------
first disable the DEBUG mode in MALMan/MALMan.cfg.

then add the following the following to /etc/httpd/conf/httpd.conf:

    LoadModule wsgi_module modules/mod_wsgi.so
    WSGIScriptAlias /MALMan /srv/MALMan/MALMan.wsgi
    <Directory /srv/MALMan>
        Order deny,allow
        Allow from all
        WSGIScriptReloading On
    </Directory>

The last snippet tells apache to load the wsgi module and serve MALMan 
(situated at /srv/MALMan) when the /MALMan directory is requested. 
For example: http://0.0.0.0/MALMan

Now restart apache to load the new config:
    systemctl restart httpd

MALMan should now be running on your server you can log in with the following 
credentials: 
    user: root@vw.be
    password: password

Running as a fastCGI app
------------------------
add this to your lighttpd config:

    fastcgi.server = (
        "/MALMan" =>
        ((
            "socket" => "/tmp/MALMan-fcgi.sock",
               "bin-path" => "/usr/local/share/webapps/MALMan/MALMan.fcgi",
            "check-local" => "disable",
            "max-procs" => 1
        ))
    )

Now restart lighttpd to load the new config:
    systemctl restart lighttpd
