---
layout: post
title:  "Making that Heroku Push"
date:   2016-12-12 11:04:00 -0500
categories: Heroku, git
---
## Making that Heroku Push
#### 1.0

I found that integrating Drupal 8 with Heroku takes a bit of effort, but is not overwhelming. The first step is to realize that Heroku is not a host like all other hosts people might be used to.
Heroku is basically a 

Database credentials are pulled from the environment. In Drupal 8 get the settings from the environment via:

  $_ENV["HEROKU_POSTGRESQL_<color>_URL"]

These values can then be used in the settings array in the following way:

  $db = parse_url($_ENV["HEROKU_POSTGRESQL_BLUE_URL"]);
  $databases['default']['default'] = [
      'database' => trim($db['path'], '/'),
      'username' => $db['user'],
      'password' => $db['pass'],
      'host' => $db['host'],
      'prefix' => 'drupal_',
      'port' => '5432',
      'namespace' => 'Drupal\\Core\\Database\\Driver\\pgsql',
      'driver' => 'pgsql',
  ];

In order to use a database dump in heroku, the dump must be put in a public space, which heroku can then reach out to in order to pull it into the instance.

  heroku pg:backups:restore 'http://<database dump location>' <database alias (sometimes includes colors!)>

Note: Heroku assumes that the pg_dump was done using the flag "--format=c". It will not accept a sql dump otherwise.

---

Nginx does not use .htaccess. In order to get rewrites to work, you must have a rewrite.conf file. An example of an nginx rewrite.conf file is:

  server_name <server dns>;

  access_log off;
  fastcgi_param SCRIPT_NAME $fastcgi_script_name;	

  location ~ \..*/.*\.php$ {
    return 403;
  }

  location ~ (^|/)\. {
    return 403;
  }

  location / {
    try_files $uri @rewrite;
  }

  location @rewrite {
    rewrite ^ /index.php;
  }

  location ~ ^/sites/.*/files/imagecache/ {
    try_files $uri @rewrite;
  }

  location ~ ^/sites/.*/files/styles/ {
    try_files $uri @rewrite;
  }

  location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
    expires max;
    log_not_found off;
  }

install heroku cli: https://devcenter.heroku.com/articles/heroku-cli

useful heroku commands:

  git push heroku - pushes the repo to the server and deploys all the files

  heroku logs - outputs log file to debug issues

  heroku ps - shows all applications in the heroku instance

  heroku run bash - logs user into the actual heroku server (via ssh)
