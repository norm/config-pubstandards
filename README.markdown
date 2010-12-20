pubstandards.com configuration
==============================

These are the config files required to run the pubstandards.com email/website
server.


Installation
------------
Instructions assume a stock Ubuntu 10.10 Maverick box.

1. Install exim as the MTA, mailman for lists and dovecot for IMAP and 
   SMTP AUTH.

        # pre-requisites
        sudo adduser --system --home /var/exim --group exim
        sudo apt-get -y install openssl libssl-dev \
            libpcre3-dev libdb4.7-dev dovecot-imapd \
            mailman listadmin
        
        # build exim from source, not apt
        cd /var/build
        wget http://ftp.exim.org/pub/exim/exim4/exim-4.72.tar.gz
        tar zxf exim-4.72.tar.gz
        cd exim-4.72
        cp src/EDITME Local/Makefile
        perl -i.bak -pe '
            s{^BIN_DIRECTORY.*}{BIN_DIRECTORY=/usr/local/bin};
            s{^CONFIGURE_FILE.*}{CONFIGURE_FILE=/etc/exim/configure};
            s{^EXIM_USER.*}{EXIM_USER=exim};
            s{^# EXIM_GROUP.*}{EXIM_GROUP=exim};
            s{^SPOOL_DIRECTORY.*}{SPOOL_DIRECTORY=/var/exim/spool};
            s{^# SUPPORT_MAILDIR.*}{SUPPORT_MAILDIR=yes};
            s{^EXIM_MONITOR.*}{# EXIM_MONITOR};
            s{^# AUTH_DOVECOT.*}{AUTH_DOVECOT=yes};
            s{^# SUPPORT_TLS.*}{SUPPORT_TLS=yes};
            s{^# TLS_LIBS=-lssl.*}{TLS_LIBS=-lssl -lcrypto};
            s{^# LOG_FILE_PATH=/var.*}{LOG_FILE_PATH=/var/log/exim/%s};
            s{^SYSTEM_ALIASES_FILE.*}{SYSTEM_ALIASES_FILE=/etc/exim/aliases};
        ' Local/Makefile
        make
        sudo make install

2. Install couchdb.

        sudo apt-get -y build-dep couchdb
        sudo apt-get -y install libicu-dev libcurl4-gnutls-dev \
            libtool xulrunner-dev
        wget http://www.mirrorservice.org/sites/ftp.apache.org//couchdb/1.0.1/apache-couchdb-1.0.1.tar.gz
        tar zxf apache-couchdb-1.0.1.tar.gz
        cd apache-couchdb-1.0.1
        xulrunner=$(echo /usr/lib/xulrunner-devel-*)
        ./configure --prefix= \
            --with-js-lib=${xulrunner}/lib \
            --with-js-include=${xulrunner}/include
        make
        sudo make install
        useradd -d /var/lib/couchdb couchdb
        chown -R couchdb: /var/lib/couchdb /var/log/couchdb
        chown -R root:couchdb /etc/couchdb
        chmod 664 /etc/couchdb/*.ini
        chmod 775 /etc/couchdb/*.d
        /etc/init.d/couchdb start
        update-rc.d couchdb defaults
        ls -d /usr/lib/xulrunner*.* > /etc/ld.so.conf.d/xulrunner.conf
        /sbin/ldconfig

3. Install nginx for front-end proxying, mini_httpd for mailman CGI 
   and Plack/Starman for the pubstandards.com site.

        sudo apt-get -y install nginx
        wget http://www.acme.com/software/mini_httpd/mini_httpd-1.19.tar.gz
        tar zxf mini_httpd-1.19.tar.gz
        cd mini_httpd-1.19
        curl https://gist.github.com/raw/747788/7d050734e8f606ccb878dc03d79cc47798f1f959/mini_httpd.patch | patch
        make
        sudo make install


Still TODO
----------
* document Plack/Starman
* checking out config and website from github
* init.d scripts for mini_httpd and plack
* crontabs for updating the site information from Flickr/Upcoming
