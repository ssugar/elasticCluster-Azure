# -*- mode: ruby -*-
# vi: set ft=ruby :

require "yaml"

_config = YAML.load(File.open(File.join(File.dirname(__FILE__), "vagrantconfig.yaml"), File::RDONLY).read)

CONF = _config

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT
sudo su
echo "Asia/Ho_Chi_Minh" > /etc/timezone
dpkg-reconfigure --frontend noninteractive tzdata
apt-get update
apt-get install curl -y
apt-get install nano -y
apt-get install nginx -y
apt-get install openjdk-7-jdk -y
cd /home/vagrant

#Get and Install Elasticsearch
curl -XGET https://raw.githubusercontent.com/ssugar/senseHCMC-ELK/master/localELK/elasticsearch-1.4.2.deb > elasticsearch-1.4.2.deb
dpkg -i elasticsearch-1.4.2.deb
update-rc.d elasticsearch defaults 95 10
curl -XGET https://raw.githubusercontent.com/ssugar/elasticCluster-Azure/master/elasticsearch.yml > elasticsearch.yml
cp /home/vagrant/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml
service elasticsearch start

#Get and Install Logstash
curl -XGET https://raw.githubusercontent.com/ssugar/senseHCMC-ELK/master/localELK/logstash-1.4.2-1.deb > logstash-1.4.2-1.deb
dpkg -i logstash-1.4.2-1.deb
service logstash restart

#Get and Install Kibana
curl -XGET https://raw.githubusercontent.com/ssugar/senseHCMC-ELK/master/localELK/kibana-latest.tar.gz > kibana-latest.tar.gz
tar -xvzf kibana-latest.tar.gz
cp -R kibana-latest /usr/share/nginx/www/kibana

#Configure Logstash
curl -XGET https://raw.githubusercontent.com/ssugar/senseHCMC-ELK/master/cookbooks/ss_logstash/files/default/logstash.conf > logstash.conf
cp /home/vagrant/logstash.conf /etc/logstash/conf.d/logstash.conf

#Install Sensor Dashboard to Kibana
curl -XGET https://raw.githubusercontent.com/ssugar/senseHCMC-ELK/master/cookbooks/ss_kibana/files/default/senseHCMCDashboard.json > senseHCMCDashboard.json
cp /home/vagrant/senseHCMCDashboard.json /usr/share/nginx/www/kibana/app/dashboards/default.json

#Restart Services
service nginx restart
service logstash restart

#Update ElasticSearch Mappings
curl -XGET https://raw.githubusercontent.com/ssugar/elasticCluster-Azure/master/updatedEsMappings.sh > updatedEsMappings.sh
sh updatedEsMappings.sh

#Configure nginx as a proxy so elasticsearch doesnt need to be public
curl -XGET https://raw.githubusercontent.com/ssugar/elasticCluster-Azure/master/nginx.conf > nginx.conf
cp /home/vagrant/nginx.conf /etc/nginx/sites-available/default

#Install the Elasticsearch-cloud-azure plugin.  Using version 2.5.1 which matches with ES version 1.4.2
/usr/share/elasticsearch/bin/plugin install elasticsearch/elasticsearch-cloud-azure/2.5.1

#Update Kibana configuration to point to elasticsearch proxy on port 80
curl -XGET https://raw.githubusercontent.com/ssugar/senseHCMC-Azure/master/kibana_config.js > kibana_config.js
cp /home/vagrant/kibana_config.js /usr/share/nginx/www/kibana/config.js
service nginx restart
SCRIPT


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
   config.vm.box = "azure"

   do_common_azure_stuff = Proc.new do |azure, override|
		override.vm.synced_folder ".", "/vagrant", disabled: true
		azure.mgmt_certificate = "az_cert.pem"
		azure.mgmt_endpoint = "https://management.core.windows.net"
		azure.subscription_id = CONF['subscription_id']
		azure.vm_image = "b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-12_04_5-LTS-amd64-server-20150204-en-us-30GB"
		azure.vm_user = "vagrant"
		azure.vm_password = CONF['password']
		azure.vm_name = "elastic"
		azure.cloud_service_name = CONF['servicename']
		azure.storage_acct_name = CONF['storagename']
		azure.vm_location = "Southeast Asia"
		azure.private_key_file = "vm_cert.key"
		azure.certificate_file = "vm_cert.pem"
   end

   config.vm.define 'first' do |cfg|
		cfg.vm.provider :azure do |azure, override|
			do_common_azure_stuff.call azure, override
			azure.vm_name = 'elastic-1'
			azure.ssh_port = "2221"
			azure.tcp_endpoints = "80"
			cfg.ssh.username = "vagrant"
			cfg.ssh.password = CONF['password']
			cfg.vm.provision "shell", inline: $script
		end
   end

   config.vm.define 'second' do |cfg|
		cfg.vm.provider :azure do |azure, override|
			do_common_azure_stuff.call azure, override
			azure.vm_name = 'elastic-2'
			azure.ssh_port = "2222"
			cfg.ssh.username = "vagrant"
			cfg.ssh.password = CONF['password']
			cfg.vm.provision "shell", inline: $script
		end
   end
   
end
