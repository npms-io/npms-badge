# Deploys

We use `pm2` to deploy `npms-badges`, install it by running `$ npm install -g pm2`. You may find the pm2 configuration file in `ecosystem.json5`.


## Setting up

Before doing the first deploy, you need to setup the server. All commands executed in the server are expected to be run with the `www` user.

- Create the `www` user on server
- Add `www` user to the list of sudoers
- Install pm2 in the server
- Setup the deploy environment by running `$ pm2 deploy ecosystem.json5 production setup` in your local machine
- Create `~/npms-badges/local.json5` in the server with the custom configuration (elasticsearch host, etc)
- Do your first deploy by running `$ pm2 deploy ecosystem.json5 production` in your local machine
- Setup logrotate by running `$ sudo pm2 logrotate -u www` on the server and then edit `/etc/logrotate.d/pm2-www` to change change `/root` to `/home/www`, weekly to daily, and from 12 days to 14 days)
- Setup pm2 to run at start by running `$ sudo pm2 startup -u www --hp "/home/www"` on the server
- Finally run `$ pm2 save` to store the running processes

### Nginx

- Install nginx in the server by running `$ sudo aptitude install nginx`
- Setup a new site called `badges` in `/etc/nginx/sites-available` with the config exemplified below
- Enable this site and finally restart nginx by running `$ sudo service nginx restart`

```
server {
  listen *:80;
  server_name badges.npms.io;

  location / {
    # Proxy to our backend
    # Need to do this ugly rewrite trickery so that url encoded parameters are left untouched
    rewrite ^ $request_uri;
    rewrite ^/(.*) $1 break;
    return 400;
    proxy_pass http://127.0.0.1:3001/$uri;

    # Do not buffer, improves performance
    proxy_buffering    off;
    proxy_buffer_size  128k;
    proxy_buffers 100  128k;

    # Fix some headers
    proxy_set_header  X-Real-IP  $remote_addr;
    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header  Host $http_host;
  }
}
```


## Deploying

Deployment is easy, just run `$ pm2 deploy ecosystem.json5 production` in your local machine.
