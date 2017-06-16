On windows machine, do

   ```
vagrant up --provider=virtualbox
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


most propably it will hang somewhere (96% ???)

If this happens, please login into the cf box (vagrant ssh cf) and do:

```
sudo su --
monit restart all
```

don't worry, if an error message occures ;)

type 
```
monit summary
```

and check whether the list looks healthy..

```
root@bosh-lite:/home/vagrant# monit summary
The Monit daemon 5.2.4 uptime: 1h 34m

Process 'nats'                      running
Process 'blobstore_nginx'           running
Process 'postgres'                  running
Process 'director'                  running
Process 'worker_1'                  running
Process 'worker_2'                  running
Process 'worker_3'                  running
Process 'worker_4'                  running
Process 'director_scheduler'        running
Process 'director_nginx'            running
Process 'health_monitor'            running
Process 'warden_cpi'                running
Process 'garden'                    running
System 'system_agent-id-bosh-0'     running

```


, maybe even repeat the monit restart all .




If everything is ok, go back to the boshlite box and restart

```
./bin/provision_cf
```
This should now get all the cloudfoundry sources from github (this takes long.. an hour or so ..?)..

# Diego
[instruction found here](https://github.com/cloudfoundry/diego-release/tree/develop/examples/bosh-lite)

```
git clone https://github.com/cloudfoundry/diego-release.git
cd diego-release/
git checkout mast
./scripts/update
SQL_FLAVOR='postgres' ./scripts/generate-bosh-lite-manifests
bosh upload release https://bosh.io/d/github.com/cloudfoundry/garden-runc-release
bosh upload release https://bosh.io/d/github.com/cloudfoundry/cflinuxfs2-rootfs-release
bosh deployment bosh-lite/deployments/diego.yml
bosh -n create release --force && bosh -n upload release && bosh -n deploy

sed -i -- 's/cflinuxfs2fs$/cflinuxfs2fs-rootfs/g' bosh-lite/deployments/diego.yml
```





