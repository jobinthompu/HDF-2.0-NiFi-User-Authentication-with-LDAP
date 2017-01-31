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