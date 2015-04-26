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
