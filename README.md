# Example static website with Docker, Nginx and Certbot

This is full project structure example of article [**How to dockerize your static website with Nginx, automatic renew SSL for domain by Certbot and deploy it to DigitalOcean?**](https://dev.to/koddr/how-to-dockerize-your-static-website-with-nginx-automatic-renew-ssl-for-domain-by-certbot-and-deploy-it-to-digitalocean-4cjc)

Published on [Dev.to](https://dev.to/koddr/how-to-dockerize-your-static-website-with-nginx-automatic-renew-ssl-for-domain-by-certbot-and-deploy-it-to-digitalocean-4cjc) @ 27 Jan 2020.

## Requirements

- Docker `19.x+`

## Usage

- Clone this repository and go to folder:

```console
git clone https://github.com/koddr/example-static-website-docker-nginx-certbot.git
cd example-static-website-docker-nginx-certbot
```

- Change `site.com` to your domain at files:

```console
./docker-compose.prod.yml
./webserver/nginx/default.conf
./webserver/nginx/site.com.conf
```

- Push configured project to your own git repository.
- Connect via SSH to your server and `git clone` your repo.
- Check configuration of `Certbot`, start the process of obtaining SSL certificate in test mode:

(optional intall make)
```console
sudo apt-get update
sudo apt-get install build-essential
```

```console
make certbot-test DOMAINS="site.com www.site.com" EMAIL=mail@site.com
```

- If you see `Congratulations!`, all okay; start the process of obtaining SSL in production mode:

```console
make certbot-prod DOMAINS="site.com www.site.com" EMAIL=mail@site.com
```

- And now, check Nginx config:

```console
make deploy-test
```

- No errors? Go your static website to production:

```console
make deploy-prod
```

- That's all!

## Basic Auth - create user and password
https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/

To create username-password pairs, use a password file creation utility, for example, apache2-utils

Verify that apache2-utils (Debian, Ubuntu) or httpd-tools (RHEL/CentOS/Oracle Linux) is installed.

```console
sudo apt-get install apache2-utils   
```

Create a password file and a first user. Run the htpasswd utility with the -c flag (to create a new file), the file pathname as the first argument, and the username as the second argument:

```console
sudo htpasswd -c /etc/apache2/.htpasswd user1 
```

Press Enter and type the password for user1 at the prompts.

Create additional user-password pairs. Omit the -c flag because the file already exists:

```console
sudo htpasswd /etc/apache2/.htpasswd user2
```

You can confirm that the file contains paired usernames and encrypted passwords:

```console
$ cat /etc/apache2/.htpasswd
user1:$apr1$/woC1jnP$KAh0SsVn5qeSMjTtn0E9Q0
user2:$apr1$QdR8fNLT$vbCEEzDj7LyqCMyNpSoBh/
user3:$apr1$Mr5A0e.U$0j39Hp5FfxRkneklXaMrr/
```

## Basic Auth - Configuring NGINX and NGINX Plus for HTTP Basic Authentication

Inside a location that you are going to protect, specify the auth_basic directive and give a name to the password-protected area. The name of the area will be shown in the username/password dialog window when asking for credentials:

```console
location /api {
    auth_basic “Administrator’s Area”;
    #...
}
```

Specify the auth_basic_user_file directive with a path to the .htpasswd file that contain user/password pairs:

```console
location /api {
    auth_basic           “Administrator’s Area”;
    auth_basic_user_file /etc/apache2/.htpasswd; 
}
```

Alternatively, you you can limit access to the whole website with basic authentication but still make some website areas public. In this case, specify the off parameter of the auth_basic directive that cancels inheritance from upper configuration levels:

```console
server {
    ...
    auth_basic           "Administrator’s Area";
    auth_basic_user_file conf/htpasswd;

    location /public/ {
        auth_basic off;
    }
}
```

## Basic Auth - Combining Basic Authentication with Access Restriction by IP Address

HTTP basic authentication can be effectively combined with access restriction by IP address. You can implement at least two scenarios:
    - a user must be both authenticated and have a valid IP address
    - a user must be either authenticated, or have a valid IP address

Allow or deny access from particular IP addresses with the allow and deny directives:

```console
location /api {
    #...
    deny  192.168.1.2;
    allow 192.168.1.1/24;
    allow 127.0.0.1;
    deny  all;
}
```

Access will be granted only for the 192.168.1.1/24 network excluding the 192.168.1.2 address. Note that the allow and deny directives will be applied in the order they are defined.

Combine restriction by IP and HTTP authentication with the satisfy directive. If you set the directive to to all, access is granted if a client satisfies both conditions. If you set the directive to any, access is granted if if a client satisfies at least one condition:

```console
location /api {
    #...
    satisfy all;    

    deny  192.168.1.2;
    allow 192.168.1.1/24;
    allow 127.0.0.1;
    deny  all;

    auth_basic           "Administrator’s Area";
    auth_basic_user_file conf/htpasswd;
}
```

## Basic Auth - Complete Example

The example shows how to protect your status area with simple authentication combined with access restriction by IP address:

```console
http {
    server {
        listen 192.168.1.23:8080;
        root   /usr/share/nginx/html;

        location /api {
            api;
            satisfy all;

            deny  192.168.1.2;
            allow 192.168.1.1/24;
            allow 127.0.0.1;
            deny  all;

            auth_basic           "Administrator’s Area";
            auth_basic_user_file /etc/apache2/.htpasswd; 
        }
    }
}
```


## License

MIT
