# Nginx

In this example, we will run a simple Nginx(1.10.X) PHP-7.0 server in a container.

We will be installing Nginx on a Ubuntu 16.04 container.

<br>

#### What you need:
 - Singularity, which you can download and install from [here](https://github.com/sylabs/singularity).
 - A text editor, like: `micro`, `vim`, or `nano`.
 - Root access.
 - A browser to test it, or you can use `w3m`, and install it by `sudo apt-get install w3m w3m-img`.


____

<br>

### To start, make the working directory:

```
$ mkdir ~/nginx
$ cd ~/nginx/
```

<br>

Now you can pull the container from the library:

```
$ singularity pull library://sylabs/examples/nginx:latest
```

<br>

Verify the container: (Optional)

```
$ singularity verify nginx_latest.sif 
Verifying image: nginx_latest.sif
Data integrity checked, authentic and signed by:
	westleyk (examples) <westley@sylabs.io>, KeyID F1E47D55A7F3A56C
```

<br>

Then change the name of the container:

```
$ mv nginx_latest.sif nginx.sif
```

<br>

Or you can build from a definition file. Click [here](#building-from-a-definition-file) or scroll down.

<br>


### Prepare our directory for the Nginx container:

To prepare our directory for Nginx, make the bind points:

```
$ mkdir -p nginx/php
```

<br>

Create the PHP file:

There are two PHP files in this repo.<br>
We will first test the `index.php` file.

The PHP file will be in `nginx/index.php`.

```
$ nano nginx/index.php
```
```
<!DOCTYPE html>
<html>
<head>
<title>Nginx container test</title>
</head>
<body>
<h1>Hello World</h1>
<?php
echo "hello world from PHP!<br>";
?>
</body>
</html>
```

Or you can download it with `wget`:

```
$ wget -O nginx/index.php https://raw.githubusercontent.com/sylabs/examples/master/http-server/nginx-php-web-server/index.php
```

<br>

### Running the container:

To start the instance:

```
$ sudo singularity instance start -B nginx/:/srv/nginx/ -B nginx/php/:/run/php/ nginx.sif nginx
```
*The `sudo` is important.*

If there are no errors, it's working correctly.


<br>

### Testing it:

To test it, open you browser to: http://localhost,<br>
or: [http://[YOUR_IP_ADDRESS]](http://111.111.1.111) 

You can also use `w3m`:

```
$ w3m localhost
# or
$ w3m <YOUR_IP_ADDRESS>
```
And you should see your web page.

<br>

### Running the test form:

Now we will run a simple PHP test form that's in this repo.

The PHP file will access a file called `data.txt`.<br>
That file will be in `/srv/nginx/data.txt`, or `nginx/data.txt` since there are linked.

<br>

To start, replace the hello world `index.php` with the PHP form:

```
$ rm nginx/index.php
$ wget -O nginx/index.php https://raw.githubusercontent.com/sylabs/examples/master/http-server/nginx-php-web-server/form_index.php
```

<br>

Now create `nginx/data.txt`:

There are two ways of doing this, of course you can just do:

```
$ touch nginx/data.txt
```

But you can also do it this way:

```
$ sudo singularity shell -B nginx/:/srv/nginx/ -B nginx/php/:/run/php/ nginx.sif 
> touch /srv/nginx/data.txt
```

<br>

Either way you do it, you need to change who owns it:

```
$ sudo chown www-data:www-data nginx/data.txt
```

Or if you're in the container:

```
> chown www-data:www-date /srv/nginx/data.txt
```

<br>

Finally, restart the instance:

```
$ sudo singularity instance stop nginx
# then startup the instance
$ sudo singularity instance start -B nginx/:/srv/nginx/ -B nginx/php/:/run/php/ nginx.sif nginx
```

Or, if you're in the container:

```
> fuser -k 80/tcp  # optional
> /etc/init.d/nginx restart
> /etc/init.d/php7.0-fpm restart
```

<br>

Now open you browser to: http://localhost,<br>
or: [http://[YOUR_IP_ADDRESS]](http://111.111.1.111) 

Or with `w3m`:

```
$ w3m localhost
# or
$ w3m <YOUR_IP_ADDRESS>
```
And you should see you web form.

<br>

### Stopping the instance:

To stop the instance, we will use the `instance stop` function:

```
$ sudo singularity instance list
[...]
$ sudo singularity instance stop nginx
```

Or if you're in the container:

```
> /etc/init.d/nginx stop
> /etc/init.dphp7.0-fpm stop
```

<br>
<br>


### Building from a definition file:

First, we need the definition file, which you can copy-paste to `nginx.def`:

```
$ nano nginx.def
```
```
Bootstrap: library
From: ubuntu:16.04

%help
Nginx 1.10.3 web server in a Ubuntu 16.04 container.

for more information,
visit: https://github.com/sylabs/examples/tree/master/http-server/nginx-php-web-server

Usage:

  setup:
  $ mkdir -p nginx/php
  $ touch nginx/index.php  # add your php script here.

  instance start:
  $ sudo singularity instance start -B nginx/:/srv/nginx/ -B nginx/php/:/run/php/ nginx.sif nginx

  or to shell:
  $ sudo singularity shell -B nginx/:/srv/nginx/ -B nginx/php/:/run/php/ nginx.sif

  you may need to run this, after shelling into the container, but before starting nginx.
  > fuser -k 80/tcp

%startscript
fuser -k 80/tcp
nginx -t
/etc/init.d/php7.0-fpm restart
/etc/init.d/nginx restart

%runscript
echo "Nginx 1.10.3 web server in a Ubuntu 16.04 container."
echo
echo "for more information,"
echo "visit: https://github.com/sylabs/examples/tree/master/http-server/nginx-php-web-server"
echo
echo "Usage:"
echo 
echo "  setup:"
echo "  $ mkdir -p nginx/php"
echo "  $ touch nginx/index.php  # add your php script here."
echo
echo "  instance start:"
echo "  $ sudo singularity instance start -B nginx/:/srv/nginx/ -B nginx/php/:/run/php/ nginx.sif nginx"
echo
echo "  or to shell:"
echo "  $ sudo singularity shell -B nginx/:/srv/nginx/ -B nginx/php/:/run/php/ nginx.sif"
echo
echo "  you may need to run this, after shelling into the container, but before starting nginx."
echo "  > fuser -k 80/tcp"
echo

%post
echo "=> installing: nginx, and php..."
apt-get -y update
apt-get -y install nginx
apt-get -y install php-fpm php-mysql

echo "=> creating nginx directory: /srv/nginx/"
mkdir /srv/nginx
mkdir /srv/nginx/php

mkdir /run/php
touch /run/php/php7.0-fpm.sock

echo "=> creating bind files..."
touch /srv/nginx/error.log
touch /srv/nginx/access.log
touch /srv/nginx/index.php
touch /srv/nginx/php7.0-fpm.log
touch /srv/nginx/php7.0-fpm.log
touch /srv/nginx/nginx.pid

echo "=> binding files..."
rm -f /var/log/nginx/error.log
ln -s /srv/nginx/error.log /var/log/nginx/error.log

rm -f /var/log/nginx/access.log
ln -s /srv/nginx/access.log /var/log/nginx/access.log

rm -f /var/www/html/index.php
ln -s /srv/nginx/index.php /var/www/html/index.php

rm -f /var/log/php7.0-fpm.log
ln -s /srv/nginx/php7.0-fpm.log /var/log/php7.0-fpm.log

rm -f /run/nginx.pid
ln -s /srv/nginx/nginx.pid /run/nginx.pid

ls /srv/nginx/
ls /srv/nginx/php/
ls /run/php/

echo "=> prepare nginx..."
mkdir /var/lib/nginx/body
mkdir /srv/nginx/body
ln -s /var/lib/nginx/body /srv/nginx/body

echo "=> preparing nginx config file..."
cat << EOF > /etc/nginx/sites-available/default 
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html/;

        index index.php index.html index.htm index.nginx-debian.html;

        server_name _;

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
        	fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    	}

	    location ~ /\.ht {
    	    deny all;
    	}

}

EOF

echo "=> testing nginx..."
nginx -t

echo "=> preparing php config file..."
cat << EOF > /etc/php/7.0/fpm/php-fpm.conf 
[global]

pid = /srv/nginx/php7.0-fpm.pid

error_log = /srv/nginx/php7.0-fpm.log

include=/etc/php/7.0/fpm/pool.d/*.conf

EOF

echo "=> done."

```

Or, just download it from Github:

```
$ wget https://raw.githubusercontent.com/sylabs/examples/master/http-server/nginx-php-web-server/nginx.def
```

<br>

Now, build the container:

```
$ sudo singularity build nginx.sif nginx.def
```

Then you should have you container (`nginx.sif`) and you don't need to download it from the library.

Now you can [Prepare our directory for the Nginx container](#prepare-our-directory-for-the-nginx-container).

<br>


____

<br>

## Troubleshooting:

 - `404 not found`:
   - No `index.php` file. Fix: make a `index.php` file in `nginx/index.php`.
<br>

 - `502 Bad Gateway`:
   - PHP not running. Fix: start PHP server.
   - Nginx does not recognize the PHP file.
<br>

 - Web page does not load:
   - Try: [http://[YOUR_IP_ADDRESS]/index.php](http://111.111.1.111) insted of http://localhost/index.php.
   - Nginx and/or PHP not running. Fix: restart the instance.
<br>

 - Web page loads, but nothing is in there:
   - PHP file is blank.
   - Nginx needs to be restarted. Fix: Stop, then start the instance.
<br>

 - The page isn’t redirecting properly:
   - Make sure the URL is typed corectly.
   - Make sure the `/index.php` is at the end of the URL.
<br>

 - Nginx wont start:
   - Incorrect, or no bind points.
   - Incorrect, or no bind directories.
   - Nginx already running elsewhere on your device.
   - Some other process is using port 80. Fix: `sudo singularity exec nginx.sif fuser -k 80/tcp`.
<br>

 - PHP wont start:
   - Incorrect, or no bind points.
   - Incorrect, or no bind directories.
   - PHP already running elsewhere on your device.
<br>

 - Nginx container won't build on `armv7l` (`32 bit`):
   - Change the `Bootstrap:` to `docker`.
<br>

 - Nginx container pulled from the library won't run on `armv7l` (`32 bit`):
   - Build from a definition file, and change the `Bootstrap:` to `docker`.
<br>

 - Error on web form: `Error: [2] filesize(): stat failed for /srv/nginx/data.txt`:
   - `/srv/nginx/data.txt` or `nginx/data.txt` dose not exist. Fix: `touch nginx/data.txt`.
<br>

 - Error on web form: `Error: [2] fopen(/srv/nginx/data.txt): failed to open stream: Permission denied`<br>
`Error: [2] fwrite() expects parameter 1 to be resource, boolean given`:
   - `srv/nginx/data.txt` owner and group not set correctly. Fix: `sudo chown www-data:www-data nginx/data.txt`.
<br>


____

<br>
<br>

