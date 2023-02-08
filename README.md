# SSL for Meteor

#### Simple built-in Meteor SSL functionality for the first time!

*or*

#### So, you wanna self-host a secure webpage developed with meteor?

## Inspiration

The original repo for this plugin can be found [here](https://github.com/nourharidy/meteor-ssl), as developed by [nourharidy](https://github.com/nourharidy). The original code hasn't been updated in some time and needs some (non-obvious) work to get it working, so this repo aims to address these issues.

## Use case

Picture this: you have worked hard to create a webpage on meteor. You have deployed your webpage on your own machine and can access it via localhost (e.g. http://localhost:1000). You have forwarded a port for your site, and made it available for users under a public IP and port combination (of the form http://255.255.255.255:1000).

Now, you are trying to secure this page via SSL, so that a user can visit it through a url of the form https://255.255.255.255:2000.

## Installation

### Step 0: Getting an SSL certificate

Using the commands below, you can generate a self-signed SSL certificate:

```sh
openssl genrsa -out localhost.key 2048
openssl req -new -x509 -key localhost.key -out localhost.cert -days 3650 -subj /CN=localhost
```

However, using a self-signed certificate will result in all your visitors receiving a scary warning message on their browser whenever they visit your website for the first time. You'll need to buy a certificate from a Certificate Authority (CA) to work around this. I used SSLs.com (not affiliated) to buy mine, although there should also be some free solutions.

At the end of this process, you'll be left with a **key** file:
```
-----BEGIN PRIVATE KEY-----
YOUR PRIVATE KEY GOES HERE
-----END PRIVATE KEY-----
```

and a **certificate** file:

```
-----BEGIN CERTIFICATE-----
YOUR CERTIFICATE GOES HERE
-----END CERTIFICATE-----
```

In my case, the **key** file had the extension *.pem*, while the **certificate** file had the extension *.crt*. These shouldn't matter, however, provided your files are of the form described above. Also, make sure the files are encoded in UTF-8, and that there are no permission issues.

### Step 1: Installing the plugin

Clone the repository:

```sh
git clone --depth=1 https://github.com/UphillD/meteor-ssl
```

Enter the plugin directory:

```sh
cd meteor-ssl
```

Install the plugin:

```sh
meteor add uphilld:ssl
```

### Step 2: Using the plugin

Place the **key** & **cert** files inside your Meteor private directory.

In your server code, add the following lines:

Import the plugin:

```
import { SSL } from 'meteor/uphilld:ssl';
```

Add the SSL function call **inside your `Meteor.startup()`** function:

```sh
SSL(
  Assets.getText("<key file>"),
  Assets.getText("<cert file>"),
  <port>);
```

Replace *\<key file\>* and *\<cert file\>* with the full name of the key and certificate files inside your private directory (without their paths).
Replace *\<port\>* with the port you wish to host the secure site on. Leave it empty for the default 443. Using the default ports requires launching meteor with sudo.

### Step 3: Deploy your site

This step depends on the way you deploy your meteor webpage. I deploy with `meteor run`, so I'll describe that process.

If you are using the default ports (80 for http, 443 for https) you need sudo:

```sudo meteor run```

If you are using non-default ports (>1024), you need to clarify the non-secure one (the plugin takes care of the secure one you typed on the previous step):

```meteor run -p 1000```

# The information below comes from the original repo.

## API
### SSL(**key**, **cert**, [**port**])
#### Server Javascript function
The **SSL()** function is used to launch the SSL functionality from the server, the SSL feature wont be present unless you use it, it must only be used inside the *server* directory.

The function has two obligatory arguments: The UTF-8 formatted string of the SSL **key** & the SSL **cert** files, respectively. The third argument is optional: Define the SSL **port** (Default: 443).

Example:
```sh
SSL(
  Assets.getText("localhost.key"),
  Assets.getText("localhost.cert"),
  443);
```

### isHTTPS()
#### Client Javascript boolean function
Returns *true* if user is using HTTPS connection

*warning*: Because this is a *client* function, this does not prevent the server from sending templates over HTTP connection, neither it prevents it from sending data over HTTP unless you prevent it from the client side.

### switchHTTPS([port])
#### Client Javascript function
This function refreshes the page after switching the browser to HTTPS. This function takes one optional argument: The SSL port previously specified by the SSL() server function (Default is 443).

Example with the iron:router:
```sh
Router.route('/', function(){
  if(isHTTPS()){
    this.route('home');
  } else {
    switchHTTPS();
  }
})
```
For the above example to work, the HTTP port must be 80, and the HTTPS port must be 443 (default).

## UI Helpers

### {{isHTTPS}}
#### Client boolean UI helper
Returns *true* if user is using HTTPS connection

This UI helper can be used in your template as a condition that returns whether or not the user is using HTTPS. This is useful to show content to the user if he is using HTTPS in certain parts of your templates.
*warning*: Because this is a *client* helper, this does not protect the template views from being sent from the server over HTTP. Using the helper, the templates/views are sent anyway but the user just can't see it unless he uses HTTPS.

Example:
```sh
{{#if isHTTPS}}
You are using HTTPS
{{else}}
You are not using HTTPS
{{/if}}
```

## Notes

* If your SSL Certificate has a password, you will be prompted with "Enter PEM passphrase" everytime the server is started.
* In order for Meteor to use port 443 for SSL (the default port), it must be started as root:
```sh
sudo meteor
```
Failing to do this can cause error `Error: listen EACCES` being thrown by dependency node-http-proxy
* In order for the *force-ssl* package to work with this package, please make sure the SSL port is 443 (default).
* You have to add the *https://* prefix to the url if you use the port number in the url. For example, assuming you chose *9000* as SSL port, this will **NOT** work:
```sh
localhost:9000
```
It will keep your browser loading forever instead of redirecting you to an HTTPS connection. To make it work, you have to add the *https//* prefix:
```sh
https://localhost:9000
```
This is why you are encouraged to use the default SSL port *443* so you can open:
```sh
https://localhost
```
To Revert the effects caused by running sudo, run this command:
```sh
sudo chown -Rh <username> .meteor/local
```

* This package does not encrypt communication between Meteor & MongoDB, to workaround this you must put MongoDB on Meteor's localhost or a server inside your secure private network.

## FAQ

### Does it support Websockets?
Yes, it encrypts both HTTP packets and Websockets (including DDP).

### Does it work with Phonegap/Cordova?

Yes, when you *run* it in development, just set the **--mobile-server** argument to the the server location preceded by the *https://* prefix & followed by the SSL port, for example if you use it on an Android device:
```sh
meteor run android-device --mobile-server=https://localhost:443
```
When you *build* the mobile application, use the same syntax with the **--server** argument, for example:
```sh
meteor build --server=https://localhost:443
```

### Does it work with server-to-server DDP connections?
Yes, just adjust **DDP.connect()** to the appropriate SSL port.

Example:
```sh
DDP.connect('https://example.com:443');
```
### Does it encrypt the connection between Meteor & MongoDB?

No it doesn't, unless MongoDB is located on localhost, all communication between Meteor & MongoDB is compromised, be careful!

### Browser shows a security warning

This is because your SSL certificate is self-signed, to prevent this you need to buy certificate signed from a Certificate Authority.

### How do i force Meteor to always use SSL?

Set the SSL port to 443 (default) and install the force-ssl smart package:
```sh
meteor add force-ssl
```

## Contact

If you have a question, a bug, or an idea, feel free to contact me:

* Send me an email: nourharidy@yahoo.com
* Send me a tweet: [@nourharidy](http://www.twitter.com/NourHaridy)
* Check out my [Github](https://github.com/nourharidy)

