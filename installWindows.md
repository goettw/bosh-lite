On windows machine, do

   ```
vagrant up
   ```

2 VMs are created

```
vagrant ssh boshlite
```

after logged in into the linux box do:

```
sudo -E apt-get update
sudo -E apt-get -y install build-essential linux-headers-`uname -r`
sudo -E apt-get -y install ruby ruby-dev git zip
sudo -E gem install bosh_cli --no-ri --no-rdoc
wget https://github.com/cloudfoundry-incubator/spiff/releases/download/v1.0.8/spiff_linux_amd64.zip
unzip spiff_linux_amd64.zip
sudo mv spiff /usr/local/bin
git clone https://github.com/goettw/bosh-lite.git
git clone https://github.com/cloudfoundry/cf-release
cd bosh-lite/
bosh target 192.168.50.4 lite
Target set to `Bosh Lite Director'
Your username: admin
Enter password: *****
Logged in as `admin'


./bin/add-route
Adding the following route entry to your local route table to enable direct warden container access. Your sudo password may be required.
  - net 10.244.0.0/19 via 192.168.50.4
  
./bin/provision_cf
```
