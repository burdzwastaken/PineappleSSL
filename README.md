# PineappleSSL
A way to force your pineapple to use SSL

##Introduction
You have a Wi-Fi Pineapple and want to harness all its power yet, everytime you log into it, you are doing so over http. Your username and passwords are being sent unencrypted over the network so the wrong people could get access to them. This guide will show you how to congigure your Pineapple so all the network traffic is encrypted.

Essentially, we are going to configure nginx (the Pineapple's server of choice) to use a self-signed certificate for https connections. To do this we need to generate a root certificate, configure openssl to act as a certificate authority, and generate an ssl certificate (signed by our root cert) for nginx to use so that clients can initiate an https connection to the Pineapple.

##Steps
###Login and get updated
First join your Pineapple's network so you can access the shell. Login to your Pineapple via SSH:

    $ ssh root@172.16.42.1

You will need to enter the root password created during setup to acceses the device. After you have access to the terminal, make sure the system is up-to-date using the following command. *For the `opkg` steps, you'll need to have your Pineapple connected to the internet. The easiest way to do so is simply connect the second radio (wlan1) to a wifi network in "client mode".*

    # opkg update
  
Once your Pineapple's package information is updated, install the `libopenssl` and `openssl_utils` packages:

    # opkg install libopenssl
    # opkg install openssl-util 

###SSL config
After you have installed the above libraries, copy over the openssl config from this repository. Enter the following commands to get to the config directory on the Pineapple, and make a copy of the current config just in case:

    # cd /etc/ssl
    # openssl.cnf openssl.conf.old

Write a new `openssl.cnf` file and paste in my openssl configuration. The pinapple ships with `vi`/`vim` and `nano`. For `vi`:

    # vi openssl.cnf

Type `i` to enter insert mode, and then paste the configuration. Type `:wq` and then `enter` to save the file. Or, replace `vi` with `nano`, paste the contents, and type `crtl+o` to write the buffer to disk. 

Alternatively, you can checkout this repository (or download the file independently) and `scp` the file to the Pineapple. On your machine, execute:

    $ git clone git@github.com:burdzz/PineappleSSL.git
    $ cd PineappleSSL
    $ scp openssl.cnf root@172.16.42.1:/etc/ssl/openssl.cnf

The configuration sets some basic values so the openssl library we just installed can process certificate signing requests using keys and a certificate we are going to generate.

###Creating the SSL/TLS certs
For this section, you'll want to be in the ssl certs directory on the Pineapple:

    # cd /etc/ssl/certs
  
Then, create the keys for both our CA and nginx,

    # openssl genrsa -aes128 -out server.key 2048
    # openssl genrsa -aes128 -out ca.key 2048
    
and remove the pass phrase on the server's private key,

    # openssl rsa -in server.key -out server.key

Now, we're going to generate our root certificate,

    # openssl req -new -x509 -days 3650 -key ca.key -out ca.pem

create a signing request for the nginx key,

    # openssl req -new -key server.key -out server.csr
    
and process the CSR using our root certificate:

    # openssl x509 -req -days 3650 -in server.csr -CA ca.pem -CAkey ca.key -set_serial 01 -out server.pem

Finally, we just need to configure nginx to use our new self-signed certificate.

###NGINX config
The nginx config is located in the nginx directory. Navigate there:

    # cd /etc/nginx

and then backup the old configuration:

    # mv nginx.conf nginx.conf.old

Copy the contents of the `nginx.conf` from this repossitory to a new conf file on the Pineapple using whatever method you used previously for the `openssl.conf` file. 

After you write the nginx configuration, you need to restart nginx in order to pickup the changes:

    # /etc/init.d/nginx restart

We've now configured nginx to use the private key and certificate we generated to facilitate ssl/tls connections with clients.

##Results
Go to your browser and type [http://172.16.42.1:1471]([http://172.16.42.1:1471), you should see a *bad request* page. Try again with https: [https://172.16.42.1:1471]([https://172.16.42.1:1471). You will now be sent to a page that warns you that your connection is not private as it is using a self-signed certificate. However, this is exactly what we want; click advanced and then proceed to your pineapples login page over SSL.
