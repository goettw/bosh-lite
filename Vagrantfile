Vagrant.configure('2') do |config|
  config.vm.define "cf" do |cf|
    cf.vm.box = 'cloudfoundry/bosh-lite'
    cf.vm.provider :virtualbox do |v, override|
      override.vm.box_version = '9000.137.0' # ci:replace
      # To use a different IP address for the bosh-lite director, uncomment this line:
      # override.vm.network :private_network, ip: '192.168.59.4', id: :local
      cf.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=777"] # ensure any VM user can create files in subfolders - eg, /vagrant/tmp
    end

    cf.vm.provider :aws do |v, override|
      override.vm.box_version = '9000.137.0' # ci:replace
      # To turn off public IP echoing, uncomment this line:
      # override.vm.provision :shell, id: "public_ip", run: "always", inline: "/bin/true"

      # To turn off CF port forwarding, uncomment this line:
      # override.vm.provision :shell, id: "port_forwarding", run: "always", inline: "/bin/true"

      # Following minimal config is for Vagrant 1.7 since it loads this file before downloading the box.
      # (Must not fail due to missing ENV variables because this file is loaded for all providers)
      v.access_key_id = ENV['BOSH_AWS_ACCESS_KEY_ID'] || ''
      v.secret_access_key = ENV['BOSH_AWS_SECRET_ACCESS_KEY'] || ''
      v.ami = ''
    end

    cf.vm.provider :vmware_fusion do |v, override|
      override.vm.box = 'cloudfoundry/no-support-for-bosh-lite-on-fusion'
      #we no longer build current boxes for vmware_fusion
      #ensure that this fails. otherwise the user gets an old box
    end

    cf.vm.provider :vmware_workstation do |v, override|
      override.vm.box = 'cloudfoundry/no-support-for-bosh-lite-on-workstation'
      #we no longer build current boxes for vmware_workstation
      #ensure that this fails. otherwise the user gets an old box
    end
  end
  
  config.vm.define "boshlite" do |boshlite|
    boshlite.vm.provision "shell", path: "bin/prep-cf-install.sh"
    boshlite.vm.provider :virtualbox do |v, override|
      override.vm.box = 'ubuntu/trusty64'

      # To use a different IP address for the bosh-lite director, uncomment this line:
      override.vm.network :private_network, ip: '192.168.50.14', id: :local
      override.vm.network :public_network
      v.memory = 6144
      v.cpus = 2
    end
  end

  config.vm.define "bosh-cli2" do |boshcli2|
    boshcli2.vm.provision "shell", inline: "apt-get update", privileged: true
    boshcli2.vm.provision "shell", inline: "apt-get -y install build-essential linux-headers-`uname -r`", privileged: true
    boshcli2.vm.provision "shell", inline: "sudo -E apt-get -y install git zip", privileged: true
    boshcli2.vm.provision "shell", inline: "wget https://s3.amazonaws.com/bosh-cli-artifacts/bosh-cli-2.0.23-linux-amd64 -O bosh;mv bosh /usr/local/bin;chmod +x /usr/local/bin/bosh", privileged: true
    boshcli2.vm.provision "shell", inline: "git clone https://github.com/goettw/bosh-lite.git", privileged: false
    boshcli2.vm.provision "shell", inline: "echo export BOSH_CLIENT=admin >> .bashrc", privileged: false
    boshcli2.vm.provision "shell", inline: "echo export BOSH_CLIENT_SECRET=admin >> .bashrc", privileged: false
    boshcli2.vm.provision "shell", inline: "bosh -e 192.168.50.4 --ca-cert <(cat bosh-lite/ca/certs/ca.crt) alias-env vbox", privileged: false
    boshcli2.vm.provision "shell", inline: "echo export BOSH_ENVIRONMENT=vbox >> .bashrc", privileged: false
    boshcli2.vm.provision "shell", inline: "git clone https://github.com/cloudfoundry/cf-deployment ~/workspace/cf-deployment", privileged: false
    boshcli2.vm.provision "shell", inline: "export STEMCELL_VERSION=$(bosh int ~/workspace/cf-deployment/cf-deployment.yml --path /stemcells/alias=default/version);bosh upload-stemcell https://bosh.io/d/stemcells/bosh-warden-boshlite-ubuntu-trusty-go_agent?v=$STEMCELL_VERSION", privileged: false
   
    boshcli2.vm.provider :virtualbox do |v, override|
      override.vm.box = 'ubuntu/trusty64'

      # To use a different IP address for the bosh-lite director, uncomment this line:
      override.vm.network :private_network, ip: '192.168.50.14', id: :local
      override.vm.network :public_network
      v.memory = 6144
      v.cpus = 2
    end
  end
end

# git clone https://github.com/cloudfoundry/cf-deployment ~/workspace/cf-deployment
#    4  cd ~/workspace/cf-deployment
#    5  export STEMCELL_VERSION=$(bosh int ~/workspace/cf-deployment/cf-deployment.yml --path /stemcells/alias=default/version)
#    6  bosh upload-stemcell https://bosh.io/d/stemcells/bosh-warden-boshlite-ubuntu-trusty-go_agent?v=$STEMCELL_VERSION
#    7  bosh update-cloud-config ~/workspace/cf-deployment/bosh-lite/cloud-config.yml
#    8  bosh -d cf deploy ~/workspace/cf-deployment/cf-deployment.yml -o ~/workspace/cf-deployment/operations/bosh-lite.yml  -v system_domain=bosh-lite.com
#    9  bosh interpolate ~/deployments/vbox/deployment-vars.yml --path /cf_admin_password
