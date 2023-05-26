# kvm-maas

ansible-playbook -i inventories/stsstack/hosts server.yml  -u ubuntu 

ansible-playbook -i inventories/stsmaas/hosts server.yml  -u ubuntu -l lxd_host

## set up juju on maas container

https://juju.is/docs/olm/maas

install latest juju
```
sudo snap install juju --classic
```
install juju 2.8
```
sudo snap install juju --classic --channel 2.8/stable
```
Create maas cloud

Create yaml maas-cloud.yaml
```
clouds:         # clouds key is required.
  maas-cloud:   # cloud's name
    type: maas
    auth-types: [oauth1]
    endpoint: http://10.4.107.156:5240/MAAS
```
then run
```
juju add-cloud --local maas-cloud maas-cloud.yaml
```
Or manually
```
$ juju add-cloud
This operation can be applied to both a copy on this client and to the one on a controller.
No current controller was detected and there are no registered controllers on this client: either bootstrap one or register one.
Cloud Types
  lxd
  maas
  manual
  openstack
  vsphere

Select cloud type: maas

Enter a name for your maas cloud: maas-cloud

Enter the API endpoint url: http://10.4.107.200:5240/MAAS/

Cloud "maas-cloud" successfully added to your local client.
You will need to add a credential for this cloud (`juju add-credential maas-cloud`)
before you can use it to bootstrap a controller (`juju bootstrap maas-cloud`) or
to create a model (`juju add-model <your model name> maas-cloud`).
```


add credential to maas cloud, need api key from admin user of maas
```
juju add-credential maas-cloud
```
Example user session:
```
This operation can be applied to both a copy on this client and to the one on a controller.
No current controller was detected and there are no registered controllers on this client: either bootstrap one or register one.
Enter credential name: maas-cloud-creds

Regions
  default

Select region [any region, credential is not region specific]: 

Using auth-type "oauth1".

Enter maas-oauth: 

Credential "maas-cloud-creds" added locally for cloud "maas-cloud".
```
bootstrap controller
```
juju bootstrap maas-cloud --no-gui --config http-proxy=http://192-168-1-0--24.maas-internal:8000/ --config https-proxy=http://192-168-1-0--24.maas-internal:8000/ --agent-version 2.8.7
```

Need to add proxy to model config
```
juju model-config apt-http-proxy=http://192-168-1-0--24.maas-internal:8000/ 
juju model-config apt-https-proxy=http://192-168-1-0--24.maas-internal:8000/ 
juju model-config snap-https-proxy=http://192-168-1-0--24.maas-internal:8000/ 
juju model-config snap-http-proxy=http://192-168-1-0--24.maas-internal:8000/ 
juju model-config http-proxy=http://192-168-1-0--24.maas-internal:8000/ 
juju model-config https-proxy=http://192-168-1-0--24.maas-internal:8000/ 
```
If k8s need to ignire http(s)-proxy and add them to containerd
```
juju config containerd no_proxy="192.168.1.0/24,127.0.0.1,localhost"
juju config containerd http_proxy="http://192-168-1-0--24.maas-internal:8000/"
juju config containerd https_proxy="http://192-168-1-0--24.maas-internal:8000/"
```
