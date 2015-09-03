# PineappleSSL
A way to force your pineapple to use SSL

##Introduction
You have a Wi-Fi Pineapple and want to harness all its power yet everytime you are logging into it this is done over http. Your username and passwords are being sent unencrypted over the network so the wrong people could get access to it. I will show you how to change this so all the network traffic is encrypted. 

##Steps
###Login and get updated
First join your pineapples network so you can access the shell. Login into your Pineapple via SSH. After you are in the terminal you won't need to us Sudo or su as you will already have root. You will need to enter your password to get into it and once you have a shell do some updates using the following command - **opkg update**. After your Pineapple is updated make sure the libopenssl and openssl_utils are installed by entering - **opkg install libopenssl** and **opkg install openssl_util**. 

###New SSL config
After you have updated and installed the above librarys you are going to need to copy across my new openssl config. Enter the follow commands to get to the directory and make a copy of the current one just in case. After you are in the file you want to paste the configuration that is in my repositories under the name **openssl.conf**. To exit vi/vim you will need to press **ESC**, type **:** and **wq** then press **Enter**.
- **cd /etc/ssl**
- **ls**
- **cp openssl.cnf openssl.conf.old**
- **rm openssl.cnf** 
- **vi openssl.cnf**

###Creating the SSL/TLS certs
You are going to want to go to the directory ssl certs by typing the command **cd /etc/ssl/certs** and then you are going to want to creating the keys by typing the following 

- **openssl genrsa -aes128 -out server.key 2048**
- **openssl genrsa -aes128 -out ca.key 2048**
- **openssl rsa -in server.key -out server.key**
- **openssl req -new -x509 -days 3650 -key ca.key -out ca.pem**
- **openssl req -new -key server.key -out server.csr**
- **openssl x509 -req -days 3650 -in server.csr -CA ca.pem -CAkey ca.key -set_serial 01 -out server.pem**

###New NGINX config
To get to the nginx config you are going to want to get to the nginx directory and then make a copy of the old configuration. After you enter vim remember how to exit by pressing **ESC**, type **:** and **wq** and press **Enter**. Once you are in the file you are going to want to paste what is in my repository named **nginx.conf**. 

- **cd /etc/nginx**
- **cp nginx.conf nginx.conf.old**
- **vi nginx.conf**

After you close the nginx.conf you are going to want to restart the nginx service by typing **/etc/init.d/nginx restart**.

##Results
After this you will want to go to your browser and type [http://172.16.42.1:1471/#] and see the result, you should get a bad request. Then try go here [https://172.16.42.1:1471/#]. See the results, you will now be sent to a page that warns you that your connection is not private as it is a self signed certificate. however click advanced and then proceed to your pineapples login page over SSL.
