# concourse-setup

# Setup Bosh Director

## Get all configs
git clone https://github.com/cloudfoundry/bosh-deployment.git

## Customize for PAAS environment
```cat config.yml << {
director_name: bosh-1
internal_cidr:  REPLACE
internal_gw: REPLACE
internal_ip: REPLACE
network_name: 'VM Network'
vcenter_dc: Datacenter
vcenter_ds: LUN01
vcenter_ip: REPLACE
vcenter_user: administrator@vsphere.local
vcenter_password: 
vcenter_templates: bosh-1-templates
vcenter_vms: bosh-1-vms
vcenter_disks: bosh-1-disks
vcenter_cluster: Cluster
dns_ip: REPLACE
ntp_ip: REPLACE
}}```

## bosh.yml 
edit bosh.yml to update dns and ntp

## create bosh env
```bosh2 create-env bosh-deployment/bosh.yml \
    --state=state.json \
    --vars-store=creds.yml \
    -o bosh-deployment/vsphere/cpi.yml \
	-l config.yml
```  
  
## Configure local alias
```$ bosh2 alias-env bosh-1 -e 10.193.56.240 --ca-cert <(bosh2 int ./creds.yml --path /director_ssl/ca)```

## Log in to the Director
```$ export BOSH_CLIENT=admin
$ export BOSH_CLIENT_SECRET=`bosh2 int ./creds.yml --path /admin_password`
```

## Query the Director for more info
```
$ bosh -e bosh-1 env
```

# Update Cloud Config
edit bosh-deployment/vsphere/cloud-config.yml to update large disks
```
bosh2 -e bosh-1 update-cloud-config ./cloud-config.yml -l config.yml
```
# Setup Concourse
```
 bosh2 -e bosh-1 upload-release https://github.com/concourse/concourse/releases/download/v3.2.1/concourse-3.2.1.tgz
 bosh2 -e bosh-1 upload-release https://github.com/concourse/concourse/releases/download/v3.2.1/garden-runc-1.6.0.tgz
 bosh2 -e bosh-1 upload-stemcell https://s3.amazonaws.com/bosh-core-stemcells/vsphere/bosh-stemcell-3421.6-vsphere-esxi-ubuntu-trusty-go_agent.tgz
```
## Generate Certs
 ```
 bosh2 int cert.yml -v dns_name=ci.haas-53.pez.pivotal.io --vars-store=gen_certs.yml
```
## deploy
 ```
 bosh2 -e bosh-1 deployment -d=concourse concourse.yml -l gen_certs.yml
 ```
