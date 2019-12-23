# k8s consul/vault/transit-app/mariadb demo
Software requirements (on your laptop, or you can also use the vagrant VM below):

```bash
gcloud git curl jq kubectl(v1.11 or greater) helm(v2.14.3 or greater) consul vault
````


## Vagrant (Optional)
You can use the vagrantfile to boot up a VM with all required software installed

0. Create VM
```bash
vagrant up
vagrant ssh
```
1. Generate ssh key and setup gcloud
```bash
cd /vagrant
ssh-keygen
gcloud auth login
```

Proceed to setup.

## Setup
0. Set your GCP creds.
https://www.terraform.io/docs/providers/google/provider_reference.html

If using TFE, use the GOOGLE_CREDENTIALS environment variable. Also the JSON credential data is required to all be on one line. Just modify in a text editor before adding to TFE.

```bash
GOOGLE_CREDENTIALS: {"type": "service_account","project_id": "klaas","private_key_id":.....}
````

1. Fill out terraform.tfvars with your values

2. plan/apply
```bash
#Make sure to set your GCP creds
terraform init;
terraform plan; 
terraform apply --auto-approve;
```

3. Go into GCP console and copy the command for  "connecting" to your k8s cluster. The command is also shown in your Terraform output.
```bash

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

gcloud_connect_command = gcloud container clusters get-credentials aklaas-connect --zone us-central1-c --project andrew-klaas
ip = 35.238.16.160
private_ip = 10.128.0.36

```
Run the command
```bash
gcloud container clusters get-credentials aklaas-connect --zone us-central1-c --project andrew-klaas
```




4. Deploy Consul/Vault/Mariadb/Python-transit-app. This takes a few minutes. We also need to pass the "private_ip" variable to the script. Grab the variable from the Terraform output. This allows the K8s Consul "datacenter" to wan join the VM consul "datacenter".
```bash
cd demo
./full_stack_deploy.sh 10.128.0.36
```
cat that script if you want to see how to deploy each of the above by hand/manually.


## UI
The demo script will now install Consul and Vault via helm. 

kubectl port forwarding is used to setup local connections to the Vault and Consul UIs. Refresh your browser tab when they initally open up. See demo/vault/vault.sh and demo/consul/consul.sh
```bash
#Consul
http://localhost:8500

#Vault
http://localhost:8200
```

Use kubectl if you'd like to inspect the deployed pods

```bas
$ kubectl get pods
NAME                                                              READY   STATUS    RESTARTS   AGE
consul-consul-connect-injector-webhook-deployment-557d4cb7wtwsn   1/1     Running   0          4m59s
consul-consul-hgc5m                                               1/1     Running   0          4m59s
consul-consul-hrxl5                                               1/1     Running   0          4m59s
consul-consul-mesh-gateway-68c66f5569-hd9rj                       1/1     Running   3          4m59s
consul-consul-mesh-gateway-68c66f5569-jsmd8                       1/1     Running   3          4m59s
consul-consul-s5znp                                               1/1     Running   0          4m59s
consul-consul-server-0                                            1/1     Running   0          4m59s
consul-consul-server-1                                            1/1     Running   0          4m58s
consul-consul-server-2                                            1/1     Running   0          4m58s
consul-consul-sync-catalog-6699948bb5-krc2c                       1/1     Running   0          4m59s
k8s-transit-app-7f54c77669-2ghfx                                  2/2     Running   1          61s
k8s-transit-app-7f54c77669-j9n9w                                  2/2     Running   1          61s
k8s-transit-app-7f54c77669-xw5n4                                  2/2     Running   1          61s
mariadb-0                                                         2/2     Running   0          3m50s
vault-0                                                           2/2     Running   0          2m57s
```

## Python Encryption as a Service App
Once the demo script completes use the following command to retrieve the public IP of the python service. Use port 5000 in your browser.

```bash
$ kubectl get svc k8s-transit-app

NAME              TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
k8s-transit-app   LoadBalancer   10.15.240.78   34.66.30.252   5000:30036/TCP   2m53s
```
For example:
```bash
http://34.68.19.26:5000
```
 


## Teardown
```bash
demo/cleanup.sh
```


