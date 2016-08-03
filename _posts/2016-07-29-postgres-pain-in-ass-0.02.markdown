---
layout: post
title:  "Vagrant PostgreSQL Drupal"
date:   2016-07-29 14:13:48 -0500
categories: Drupal postgresql vagrant
---
## Oh PostgreSQL, what a pain in the ass you are
#### 0.02

using vaprobash, uncomment nginx & postgresql

connect to postgre while on local: sudo -u postgres psql

view all users \du

give root user superuser priv: ALTER USER root SUPERUSER CREATEDB;

allow postgres to listen to all connections: 

###### Open up remote access

    sudo nano /etc/postgresql/9.5/main/pg_hba.conf

    host    all             all             all                     md5

    sudo nano /etc/postgresql/9.5/main/postgresql.conf

    listen_addresses='*'

###### Restart nginx

    sudo /etc/init.d/nginx restart

###### Restart postgres

    sudo /etc/init.d/postgresql reload

###### Dump into postgres

    psql -U [user name] -h [ip addr] [database name] < [file name]