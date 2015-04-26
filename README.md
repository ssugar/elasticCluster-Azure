#elasticCluster-Azure#

##Description##
Setting up an elasticsearch cluster on azure using vagrant.

===================================
##Setup to use this##
There are a number of steps you need to complete in order to run this VM on an Azure subscription.

###Install vagrant-azure plugin###
    vagrant plugin install vagrant-azure

###Install openssl###
    chocolatey install openssl.light -y
    
###Set up and Upload Azure Management Certificate###
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout az_cert.pem -out az_cert.pem
    openssl x509 -inform pem -in az_cert.pem -outform der -out az_cert.cer

Upload the resultant az_cert.cer file to Azure --> Manage Subcription --> Management Certificate --> Upload
	
###Set up VM Management Certificate###
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout vm_cert.key -out vm_cert.pem
  
###Create Azure dummy box###
    vagrant box add azure https://github.com/msopentech/vagrant-azure/raw/master/dummy.box

###Create vagrantconfig.yaml file###
In the same folder as your Vagrantfile, create a file called vagrantconfig.yaml.  That file should look like:

    subscription_id: your_azure_subscription_ID
	password: password_for_the_VMs_vagrant_account
	servicename: desired_cloud_service_name
	storagename: desired_storage_account_name

###Bring VMs up###
    vagrant up --provider=azure
	
###Configure Load Balancing###
You may notice in the vagrantfile that only the first VM gets a TCP port 80 endpoint.  To complete load balancing across the set, do the following:
+ Log into the Azure Management Portal
+ Go to the Endpoints screen for the first VM
+ Configure the existing TCP port 80 endpoint as a Load Balanced Set
+ Go to all other VMs Endpoint screen
+ Add and Endpoint to an Existing Load Balanced Set

**Future Improvement** - Use the Azure Service Management REST API to configure the load balanced set programatically.

###Set Up Elasticsearch Cluster Discovery in Azure###
The elasticsearch nodes won't talk to each other without some help.  

**Future Improvement** - Integrate the following into the deployment process.

######To be done on each VM######
+ Upload the az_cert.pem file to the cloud service
+ Convert the az_cert.pem file to .pkcs12 with the following command:
  * You will be prompted for a password, it will be used in the elasticsearch.yml file
    openssl pkcs12 -export -in az_cert.pem -out az_keystore.pkcs12 -name azure -noiter -nomaciter
+ Upload the resultant .pkcs12 file to each VM, remember the path
+ Insert the following into the elasticsearch.yml file just above the Slow Log section:
    cloud:
        azure:
             subscription_id: XXXXXXXX-XXXX-XXXX-XXXXX-XXXXXXXXXXXX
             service_name: desired_cloud_service_name
             keystore: /path/to/az_keystore.pkcs12
             password: password_set_when_creating_pkcs12_file

    discovery:
        type: azure
+ Restart elasticsearch
    service elasticsearch restart
+ Check elasticsearch logs to see if this works, or if there are errors:
    tail -f /var/log/elasticsearch/elasticsearch.log


