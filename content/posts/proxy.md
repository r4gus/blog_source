+++
title = "HTTP/2 Reverse Proxy using Apache"
date = 2024-01-22T23:45:00+01:00
math = true
author = "David P. Sugar"
cover = ""
tags = ["apache", "reverse-proxy", "proxy"]
keywords = ["apache", "reverse-proxy", "proxy"]
description = "How to setup a reverse proxy using Apache"
showFullContent = false
readingTime = false
hideComments = false
+++

This is a short tutorial on how to configure a reverse-proxy using apache, that translates `http/1.1` requests into `http/2` requests.

Imagine you have a server that only supports `http/2` but a client that wants to connect to the server supports only `http/1.1`. In such a case it would be handy to have some kind of mechanism that translates between client and server. A reverse proxy. For this tutorial we are going to configure a Apache web-server on Ubuntu 22.04 that acts as a reverse proxy to exchange messages between a `http/2` server and and `http/1.1` client.

## 1. Install Apache on Ubuntu 22.04

Runt the following commands to install the Apache web-server on your system:

```
sudo apt update && sudo apt upgrade && sudo apt install apache2
sudo systemctl enable apache2
sudo systemctl start apache2
```

You can verify the status of Apache using `systemctl status apache2`.

> NOTE: If you get an error when restarting apache2, try to comment out `Listen 80` in `/etc/apache2/ports.conf`. The Apache server will not start if one of the ports in `/etc/apache2/ports.conf` is already bound by another process.

## 2. Enable Apache Modules

Next wee need to enable some Apache modules required for the reverse-proxy.

```
sudo a2enmod proxy proxy_http proxy_http2
```

After you've enabled the required modules, you must restart the Apache server so the changes can take effect.

```
sudo systemctl restart apache2
```

To check which modules have been loaded, run `sudo apachectl -M`.

## 3. Disable default configuration

Next we need to disable the default configuration. Otherwise, you will only get a default HTML site when connecting to the Apache server.

```
sudo a2dissite 000-default.conf
```

## 4. Create a reverse-proxy configuration

Create a new file `/etc/apache2/sites-available/reverse_proxy.conf` and add the following:

```
LoadModule http2_module modules/mod_http2.so

<VirtualHost *:8080>
    #ProxyPreserveHost On
    
    ProxyPass / "h2c://192.168.56.2:7777/"
    ProxyPassReverse / "h2c://192.168.56.2:7777/"
</VirtualHost>
```

The configuration includes the following:
* `*:8080` - Listen on port `8080`.
* `ProxyPass` - Define the target address for traffic redirections. In our case this will forward incoming requests to the server located at `192.168.56.2:7777`. The [`h2c`](https://httpd.apache.org/docs/2.4/mod/mod_proxy_http2.html) tells Apache to proxy each http/1.1 request to the backend using http/2 via TCP (no TLS).
* `ProxyPassReverse` - This rewrites the original location, content location, and URI https response headers of the backend server with the proxy's information.
* `ProxyPreserveHost` - This will pass the original Host header to the backend server.

Also, you need to modify `/etc/apache2/ports.conf` and add the port specified above.

> NOTE: Please adjust the ip address of the backend server and the ports as needed.

Finally, enable the new configuration.

```
sudo a2ensite reverse_proxy.conf
sudo systemctl reload apache2
```

You should now be able to access the backend server through your reverse proxy. In the example below I use curl to send a GET request to a [Open5GS](https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/) NRF listening on `192.168.56.2:7777`. Keep in mind that the Apache server runs on localhost and that the reverse proxy listens on port `8080` in our case.

```
$ curl -X GET http://127.0.0.1:8080/nnrf-nfm/v1/nf-instances
{
    "_links": {
        "items": [{
            "href": "http://192.168.56.2:7777/nnrf-nfm/v1/nf-instances/xyz"
        }],
        "self": {
            "href": "http://192.168.56.2:7777/nnrf-nfm/v1/nf-instances
        }
    }
}
```
