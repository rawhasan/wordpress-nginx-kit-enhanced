# Nginx Optimizations from the Guide

## User & Worker Process

Get the number of CPU cores your server has available:

```grep processor /proc/cpuinfo | wc -l```

Get your server’s open file limit:

```ulimit -n```

The `worker_processes` directive determines how many workers to spawn per server. The general rule of thumb is to set this to the number of CPU cores your server has available. In my case, this is `1`.

`worker_connections` should be set to your server’s open file limit. This tells Nginx how many simultaneous connections can be opened by each worker_process. Therefore, if you have two CPU cores and an open file limit of 1024, your server can handle 2048 connections per second.

The `multi_accept` directive should be uncommented and set to `on`. This informs each `worker_process` to accept all new connections at a time, opposed to accepting one new connection at a time.

## Http Block

In `http` block. The first directive to add is `keepalive_timeout`. The `keepalive_timeout` determines how many seconds a connection to the client should be kept open before it’s closed by Nginx. This directive should be lowered, as you don’t want idle connections sitting there for up to 75 seconds if they can be utilized by new clients. I have set mine to `15`. You can add this directive just above the `sendfile on;` directive.

For security reasons, you should uncomment the `server_tokens` directive and ensure it is set to `off`. This will disable emitting the Nginx version number in error messages and response headers.

Underneath `server_tokens` add the following line to set the maximum upload size you require in the WordPress Media Library:

```client_max_body_size 64m;```

I chose a value of `64m` but you can increase it if you run into issues uploading large files.

## Gzip Compression

Uncomment the `gzip_proxied` directive and set it to `any`, which will ensure all proxied request responses are gzipped.

Uncomment the `gzip_comp_level` and set it to a value of `5` (ChatGPT Suggests `6` as Standard). This controls the compression level of a response and can have a value in the range of 1 – 9. Be careful not to set this value too high, as it can have a negative impact on CPU usage.

Uncomment the `gzip_types` directive, leaving the default values in place. This will ensure that JavaScript, CSS, and other file types are gzipped in addition to the HTML file type which is always compressed by the gzip module.

**Restart Nginx**
```
sudo nginx -t
sudo service nginx restart
```



## Run WordPress as the Server User

Open the Nginx configuration file: 

```sudo nano /etc/nginx/nginx.conf```

Set the `user` to the username that you’re currently logged in with. This will make managing file permissions much easier in the future.

Open the default pool configuration file:
```
sudo nano /etc/php/8.3/fpm/pool.d/www.conf
```

Change the following lines, replacing `www-data` with your username:
```
user = YOUR-USERNAME
group = YOUR-USERNAME
```
```
listen.owner = YOUR-USERNAME
listen.group = YOUR-USERNAME
```




## WordPress maximum upload size
You should adjust your `php.ini` file to increase the WordPress maximum upload size. Both this and the `client_max_body_size directive` within Nginx must be changed for the new maximum upload limit to take effect.

Open your php.ini file:
```
sudo nano /etc/php/8.3/fpm/php.ini
```

Change the following lines to match the value you assigned to the client_max_body_size directive when configuring Nginx:
```
upload_max_filesize = 64M
```
```
post_max_size = 64M
```



## OPcache
Enable the OPcache file override setting. When this setting is enabled, OPcache will serve the cached version of PHP files without checking if the file has been modified on the file system, resulting in improved PHP performance.

Open your php.ini file:
```
sudo nano /etc/php/8.3/fpm/php.ini
```

Hit `CTRL + W` and type `file_override` to locate the line we need to update. Now uncomment it (remove the semicolon) and change the value from zero to one:
```
opcache.enable_file_override = 1
```

Before restarting PHP, check that the configuration file syntax is correct:
```
sudo php-fpm8.3 -t
```

```
sudo service php8.3-fpm restart
```

Now that Nginx and PHP have been installed, you can confirm that they are both running under the correct user by issuing the `htop` command:

If you hit `SHIFT + M`, the output will be arranged by memory usage which should bring the `php-fpm` processes into view. If you scroll to the bottom, you’ll also find a couple of nginx processes.

Both processes will have one instance running under the root user. This is the main process that spawns each worker. The remainder should be running under the username you specified.

If not, go back and check the configuration, and ensure that you have restarted both the Nginx and PHP-FPM services.








