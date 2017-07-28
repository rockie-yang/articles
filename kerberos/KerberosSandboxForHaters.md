# Kerberos Sandbox for Haters

Kerberos is not a new techolodge. There are also varies information could be found on internet. 

Still when I get a task to work on Kerberos, I found myself stand in a dark rain forest, alone. I hope these article helps a bit if you are the same as me standing in a dark rain forest now.

## Terms

| Term              | Description |
|------------------:|------------:|
|principals         |name/password pairs (along with their expiration time and related options)
|Ticket             | granted by Kerberos that to indicate a user accessing a service
## So what is Kerberos

* Authentication, determine who you are.
* Authorization, determin what you can do.

## Get a sandbox Kerberos server

    docker run -d -p 88:88/udp knockdata/kerberos-sandbox
    
## Get a sandbox Kerboros client

    
## Create your own Kerberos sandbox

### Create a docker container

Clone a sandbox github repo    

    git clone ...

Build a base image for the Kerberos sandbox
    
    docker build -t kerberos-sandbox .

Start the Kerberos sandbox container
    
    docker run -d -p 88:88/udp --name kerberos-sandbox kerberos-sandbox

Connect to Kerberos sandbox container
    
    docker exec -it kerberos-sandbox bash
    
### Configure /etc/hosts

Add following line to /etc/hosts. example.com will resolve to 127.0.0.1 (localhost)
    
    127.0.0.1	example.com


### Install Kerberos KDC (Key Distribution Center)

run following command to install Kerberos 

    apt-get install -y krb5-kdc

Answering with following configuration when prompt

    Realm: EXAMPLE.COM
    domain: example.com

### Install Kerberos Admin Server

    apt-get install -y krb5-admin-server

### Create newrealm (I set the password to admin)

Run following command to create a new realm
    
    krb5_newrealm

Set the password, when prompt `Enter KDC database master key:`. I just set the super secure password `admin`. TODO, when to use this password


Check if process for Kerberos is started properly. Run following command

    ps -ef
    
It shall have two process running. Like this.
    
    root        24     0  0 10:19 ?        00:00:00 /usr/sbin/krb5kdc -P /var/run/krb5kdc.pid
    root        31     0  0 10:19 ?        00:00:00 /usr/sbin/kadmind -P /var/run/kadmind.pid


    
### Add admin principal (I set the password to rockie)

Run following command 

    kadmin.local
    
Add a principal like `rockie/admin` and quit `kadmin.local`

    Authenticating as principal root/admin@EXAMPLE.COM with password.
    kadmin.local:  addprinc rockie/admin
    WARNING: no policy specified for rockie/admin@EXAMPLE.COM; defaulting to no policy
    Enter password for principal "rockie/admin@EXAMPLE.COM":
    Re-enter password for principal "rockie/admin@EXAMPLE.COM":
    Principal "rockie/admin@EXAMPLE.COM" created.
    kadmin.local:  quit

Add admin principal to  Access Control List (ACL)

    echo "rockie/admin@EXAMPLE.COM        *" >> /etc/krb5kdc/kadm5.acl

restart the krb5-admin-server for the new ACL to take affect:

    service krb5-admin-server restart
    service krb5-kdc restart

### Add a service principal

Run following command to create a principal `HTTP/www.example.com`

    kadmin -q "addprinc -randkey HTTP/www.example.com"

The `-randkey` option of addprinc specifies that the encryption key should be chosen at random instead of being derived from a password. Services normally authenticate using a keytab, so have no need for a password.

### Get keytab for the service principal

Run the following command to add the keytab for `HTTP/www.example.com` to file `/etc/http.keytab`

    kadmin -q "ktadd -k /etc/http.keytab HTTP/www.example.com"


__TODO Kerbernized service__

## Acknoledgement

* [Client/Server Hello World in Kerberos, Java and GSS](http://thejavamonkey.blogspot.dk/2008/04/clientserver-hello-world-in-kerberos.html)
* [Explain like I’m 5: Kerberos](http://www.roguelynn.com/words/explain-like-im-5-kerberos/)
* [Hadoop and Kerberos: The Madness beyond the Gate](https://steveloughran.gitbooks.io/kerberos_and_hadoop/)
* [Ubuntu Server Guide Kerberos](https://help.ubuntu.com/lts/serverguide/kerberos.html)
* [MIT Kerberos installation on Debian](https://debian-administration.org/article/570/MIT_Kerberos_installation_on_Debian)
* [Create a service principal using MIT Kerberos](http://www.microhowto.info/howto/create_a_service_principal_using_mit_kerberos.html)
* [Add a host or service principal to a keytab using MIT Kerberos](http://www.microhowto.info/howto/add_a_host_or_service_principal_to_a_keytab_using_mit_kerberos.html)


http://thejavamonkey.blogspot.dk/2008/04/clientserver-hello-world-in-kerberos.html
http://www.roguelynn.com/words/explain-like-im-5-kerberos/
https://hc.apache.org/httpcomponents-client-ga/tutorial/html/authentication.html