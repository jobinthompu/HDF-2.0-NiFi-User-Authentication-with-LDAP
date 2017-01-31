# HDF-2.0-NiFi-User-Authentication-with-LDAP

##Short Description:

Configuring NiFi to use LDAP Authentication via Ambari

##Introduction

By integrating with LDAP, username/password authentication can be enabled in NiFi. This tutorial provides step by step instructions to setup NiFi - LDAP Authentication via Ambari (Using Knox Demo Ldap Server)

##Prerequisite

1) Assuming you already have HDF-2.x Installed on your VM/Server, Ambari, NiFi is up and running with out security.

If not, I would recommend "Ease of Deployment" section of [this](https://community.hortonworks.com/articles/57980/hdf-20-apache-nifi-integration-with-apache-ambarir.html) article to install it [You can also follow this [article](https://community.hortonworks.com/articles/56849/automate-deployment-of-hdf-20-clusters-using-ambar.html) for Automated installation of HDF cluster or refer hortonworks.com for detailed steps]

##Setting up Demo LDAP Server

1) As HDF and HDP cannot co-exist on a single node, lets download [knox](http://mirror.cogentco.com/pub/apache/knox/0.11.0/knox-0.11.0.zip) zip file from apache for this tutorial for easily setting up an ldap server. Execute below steps for the same after establishing ssh connectivity to the VM/Server (name of my host is node1):

```
# ssh node1
# mkdir /opt/knox/
# cd /opt/knox/
# wget http://mirror.cogentco.com/pub/apache/knox/0.11.0/knox-0.11.0.zip
# unzip knox-0.11.0.zip
# /opt/knox/knox-0.11.0/bin/ldap.sh start
```

2) Make sure ldap server is started and running on port 33389 on your server

```
# lsof -i:33389
    OR
# netstat -anp | grep 33389
```

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/ldap_verify.jpg)

3) Below credentials are part of knox demo ldap we just started. We can use any of the users to login to NiFi after integration.

```
tom/tom-password
admin/admin-password
sam/sam-password
guest/guest-password
```

##Configuring NiFi for LDAP Authentication via Ambari


1) Login to Ambari UI in the server URL, Click on the NiFi service à and then click on Config tab, expand “Advanced nifi-ambari-ssl-config ” section, update configuration as below:

```
Initial Admin Identity : uid=admin,ou=people,dc=hadoop,dc=apache,dc=org
Enable SSL? : {click check box} 
Key password : hadoop 
Keystore password : hadoop
Keystore type : JKS
```

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/Advanced_nifi_ambari_ssl_config.jpg)

2) Enter Below as the Truststore and DN configurations :

```
Truststore password : hadoop
Truststore type : JKS 
NiFi CA DN prefix : CN= 
NiFi CA DN suffix : , OU=NIFI
```

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/Truststore_and_DN.jpg)


3) Provide the configuration as below for Node Identities and keystore details:

```
NiFi CA Force Regenerate? : {click check box} 
NiFi CA Token : hadoop  
Node Identities :
	<property name="Node Identity 1">CN=node1, OU=NIFI </property>
	<property name="Node Identity 2">CN=node2, OU=NIFI </property>
	<property name="Node Identity 3">CN=node3, OU=NIFI </property>

```

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/Node_Identity.jpg)


4) In the Ambari UI, choose NiFi service and select config tab. We have to update two set of properties, in the “Advanced nifi-properties ” section update nifi.security.user.login.identity.provider as ldap-provider

```
nifi.security.user.login.identity.provider=ldap-provider
```

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/ldap_provider.jpg)

5) Now in the “Advanced nifi-login-identity-providers-env ” section, update the “Template for loginidentity-providers.xml “ property with below configurations just above </loginIdentityProviders>

```
<provider>
<identifier>ldap-provider</identifier>
<class>org.apache.nifi.ldap.LdapProvider</class>
<property name="Authentication Strategy">SIMPLE</property>
<property name="Manager DN">uid=admin,ou=people,dc=hadoop,dc=apache,dc=org</property>
<property name="Manager Password">admin-password</property>
<property name="TLS - Keystore">/usr/hdf/current/nifi/conf/keystore.jks</property>
<property name="TLS - Keystore Password">hadoop</property>
<property name="TLS - Keystore Type">JKS</property>
<property name="TLS - Truststore">/usr/hdf/current/nifi/conf/truststore.jks</property>
<property name="TLS - Truststore Password">hadoop</property>
<property name="TLS - Truststore Type">JKS</property>
<property name="TLS - Client Auth"></property>
<property name="TLS - Protocol">TLS</property>
<property name="TLS - Shutdown Gracefully"></property>
<property name="Referral Strategy">FOLLOW</property>
<property name="Connect Timeout">10 secs</property>
<property name="Read Timeout">10 secs</property>
<property name="Url">ldap://node1:33389</property>
<property name="User Search Base">ou=people,dc=hadoop,dc=apache,dc=org</property>
<property name="User Search Filter">uid={0}</property>
<property name="Authentication Expiration">12 hours</property>
</provider>
```
![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/loginIdentityProviders.jpg)

6) Once All properties are updated, click save and when prompted, click restart.

7) Once restarted, you can try connecting to nifi URL, you should see the below screen, enter credentials as below for admin user the configured Initial Admin Identity and click LOG IN

```
https://node1:9091/nifi/     --> in my case host is node1
 
admin/admin-password
```
![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/nifi-login.jpg)

8) You should be able to login as Admin user for NiFi and should see the below UI:

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/NiFi_UI.jpg)

##Adding a User and Providing Access to UI

1) Let us go ahead and create a user jobin in ldap so that we can give access for him to NiFi UI.

2) Edit the users.ldif file with below entry in the knox/conf directory and restart the server:

```
# vi /opt/knox/knox-0.11.0/conf/users.ldif
```
Add below entry to the end of the file

```
# entry for sample user jobin
dn: uid=jobin,ou=people,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass:person
objectclass:organizationalPerson
objectclass:inetOrgPerson
cn: jobin
sn: jobin
uid: jobin
userPassword:jobin-password
```

3) Once added lets stop and start the ldap server:

```
# /opt/knox/knox-0.11.0/bin/ldap.sh stop
# /opt/knox/knox-0.11.0/bin/ldap.sh start
```

4) While logged in as admin on the nifi UI, Lets us add a user jobin with below id by clicking '+ user' button on top right 'users' menu like below:

```
uid=jobin,ou=people,dc=hadoop,dc=apache,dc=org
```

Enter the above value and click OK.
![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/user_add.jpg)

5) Now close the users window and click to open 'policies' window on the management menu on the top right corner below 'users' menu. click "+user" button on right top corner, on the pop up, enter jobin and select the user and click OK.

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/jobin_policy.jpg)

6) Once policy added, it will look like below:

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/policy_done.jpg)

7) Now you may log out as admin and provide below credentials to login as 'jobin' user,

```
jobin/jobin-password
```

8) you should be able to login and view the UI, but wont have privilege to add anything to the canvas. (as jobin is given only read access) you may login back as admin and give required access.

![alt tag](https://github.com/jobinthompu/HDF-2.0-NiFi-User-Authentication-with-LDAP/blob/master/images/jobin_loggedin.jpg)


###This completes the tutorial, You have successfully:

- Installed and Configured HDF 2.0 on your server.

- Downloaded and started knox Demo Ldap Server

- Configured NiFi to use Knox Ldap to Authenticate users where NiFi Initial Admin is from Ldap.

- Restarted NiFi and verified access for admin user in NiFi UI.

- Created a new user jobin in ldap, added him to NiFi user list and gave read access.

- Verified access for user jobin

Thanks,

Jobin George