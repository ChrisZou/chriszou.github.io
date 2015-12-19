---
layout: post
title: Nginx with Unicorn to serve your rails app
date: 2015-12-19T12:23:59+08:00
comments: true
---
# Nginx with Unicorn to serve your rails app
This is currently the most popular solution to serve rails app and is used by many big companies like github, tweet, 37signals and so on. Also, it is very easy to setup in this way.

## Config unicorn

### Install gem unicorn:
Put `gem ‘unicorn’` in Gemfile
then `bundle install`

### Edit unicorn config file
Vim `config/unicorn.rb` under your rails app root dir.  
content:

```
# The ONLY too things you should change, if you don’t need any specialty.
app_name = "your_app_name"
number_of_app_instances = 2
# Set the working application directory. This should be your rails app root dir, not the public dir
app_root = File.expand_path(File.dirname(__FILE__) + '/..')
working_directory app_root

# File to store unicorn pid
# pid "/path/to/pids/unicorn.pid"
pid "#{app_root}/tmp/pids/unicorn.pid"

# Path to logs
# stderr_path "/path/to/log/unicorn.log"
# stdout_path "/path/to/log/unicorn.log"
stderr_path "#{app_root}/log/unicorn.log"
stdout_path "#{app_root}/log/unicorn.log"

# Unicorn socket
listen "/tmp/unicorn.#{app_name}.sock"

# Number of processes
worker_processes number_of_app_instances

# Time-out
timeout 30
```

## Nginx config

### Install nginx
```
sudo apt-get update
sudo apt-get install nginx
```
Default nginx config file would be located in `/etc/nginx`

### Nginx config file
Add a file in sites_available and put content:

```
upstream app {
    # Path to Unicorn SOCK file, as defined previously
    server unix:/tmp/unicorn.your_app_name.sock fail_timeout=0;
}

# configure server for your app
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    root /path/to/your/rails/root/public;

    location ^~ /assets/ {
        gzip_static on;
        expires max;
        add_header Cache-Control public;
    }

    try_files $uri @app;
    location @app {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://app;
    }
}
```

### Make a soft link in `sites-enabled` to the above nginx config file

### Reload nginx
`sudo service nginx restart`

## Start serving

### Compile assets: rake assets:precompile
If you only want to server your app in development environment, you don't need this step.

### Start unicorn in production
`bundle exec unicorn -c config/unicorn.rb -E production`
