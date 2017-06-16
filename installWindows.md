On windows machine, do

   ```
vagrant up --provider=virtualbox
   ```

2 VMs are created 

```
vagrant ssh boshlite 
```
*I do this with the git bash Application. Windows Powershell does not find a ssh-client (in my case)*
once logged in into the linux box do:

```
cd bosh-lite/
bosh target 192.168.50.4 lite
Target set to `Bosh Lite Director'
Your username: admin
Enter password: *****
Logged in as `admin'


./bin/add-route
Adding the following route entry to your local route table to enable direct warden container access. Your sudo password may be required.
  - net 10.244.0.0/19 via 192.168.50.4
  
sudo ./bin/provision_cf
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
git checkout master
./scripts/update
SQL_FLAVOR='postgres' ./scripts/generate-bosh-lite-manifests
bosh upload release https://bosh.io/d/github.com/cloudfoundry/garden-runc-release
bosh upload release https://bosh.io/d/github.com/cloudfoundry/cflinuxfs2-rootfs-release
bosh deployment bosh-lite/deployments/diego.yml

sed -i -- 's/cflinuxfs2fs$/cflinuxfs2fs-rootfs/g' bosh-lite/deployments/diego.yml

sudo bosh -n create release --force && bosh -n upload release && bosh -n deploy


```





