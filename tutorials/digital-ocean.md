# Installing on Digital Ocean

This tutorial will show you how to setup your Lab 5 application on Digital
Ocean. Prior to this tutorial you should have already installed Lab 4 following
the instructions we gave then.

## Configure nginx

We need to configure your web server, nginx, so that it has a new server block for this lab. Let's assume it is called `lab5.yourdomain.com`. Substitute your own domain as needed.

Navigate to the directory where nginx sites are configured:

```
cd /etc/nginx/sites-available
```

Create a new file there:

```
sudo touch lab5.yourdomain.com
```

Edit this file with the editor of your choice. Be sure to use sudo. Put the following there:

```
server {
  listen 80;
  server_name lab5.yourdomain.com;
  root /var/www/lab5.yourdomain.com;
  index index.html;
  default_type "text/html";
  location / {
    # Serve static files from the root location above.
    try_files $uri $uri/ /index.html;
  }
  location /api {
    # Serve api requests here. This will connect to your node
    # process that is running on port 3001.
    proxy_pass http://localhost:3001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```

This will setup nginx so that any requests to `lab5.yourdomain.com` go to `/var/www/lab5.yourdomain.com`, except if the request starts with `/api`. In that case, the request will be routed to your Node server running on port 3001.

**Note that we are using port 3001 for this lab.** It must be different than
the port that Lab 4 is running on, and likewise different from the port for your
creative project. This must match whatever you setup in `src/server.js` and in
`vue.config.js`.

The next step is to link this into the `sites-enabled` directory:

```
cd ../sites-enabled
sudo ln -s ../sites-available/lab5.yourdomain.com .
```

You should be able to see that this is working properly:

```
ls -al
```

You will see something like this:

```
lrwxrwxrwx 1 root root   34 Mar 14 01:20 lab5.mydomai.com -> ../sites-available/lab5.mydomain.com
```

This shows that the file is linked correctly. You can also try using `cat` to view the file and make sure it looks right.

The final step is to reload nginx so that this configuration takes effect:

```
sudo nginx -s reload
```

If nginx fails to reload properly, then most likely you have a syntax error in the configuration for a host, or your soft link (from `ln -s`) is incorrect.

## Use nvm

```
nvm use stable
```

## Clone your code

Clone your repository into your **home directory**. For example:

```
cd ~/
git clone
cd lab5
```

## Fix your node configuration

We setup node for your local machine. There is one thing we need to change in
`server.js`. Make sure you configure multer so that it stores files in your
public web server directory. For example:

```
const upload = multer({
  dest: '/var/www/lab5.mydomain.com/images/',
  limits: {
    fileSize: 10000000
  }
});
```

You need to change `dest` so that it points to the correct directory. You should
only need to change the host name if you have been following my configuration.

## Setup Node

Now install the packages you will need:

```
npm install
```

This will find all the packages in package.json and install them into this
directory. Make sure you checked this file into your repository.

Make sure your server runs:

```
node server.js
```

You just want to make sure it starts without errors. Kill it after you check
this with `control-c`.

## Setup your public files

In your project directory, run:

```
npm run build
```

This will build the front end for your project and put it into the `dist` folder.
Now copy everything in the `dist` folder to `/var/www`. For example:

```
sudo mkdir /var/www/lab5.mydomain.com
sudo chown zappala /var/www/lab5.mydomain.com
cp -rp dist/* /var/www/lab5.mydomain.com/
```

Be sure to use your username instead of `zappala`.

## Run node forever

You can run your server with:

```
forever start server.js
```

You can check its status with:

```
forever list
```

Note that this will show the log file where output from `server.js` will be stored. This will come in handy!

You can also use:

```
forever stop 0
```

to stop forever job number 0.

## Testing

Everything should be setup. You should be able to browse to your site, e.g `lab5.mydomain.com` and have the site work.

## Debugging

If your site is not working, you should (1) look at the JavaScript console, (2)
look at the Network tab in Developer Tools, (3) look at the output of the
forever log (see above).
