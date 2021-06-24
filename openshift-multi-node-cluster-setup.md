* yum update
* yum install -y wget git zile nano net-tools docker-1.13.1 bind-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct openssl-devel httpd-tools NetworkManager python-cryptography python3-pip python-devel python-passlib java-1.8.0-openjdk-headless "@Development Tools"
* yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
* sed -i -e "s/^enabled=1/enabled=0/" /etc/yum.repos.d/epel.repo
* systemctl start NetworkManager
* systemctl enable NetworkManager
* pip3 install ansible==2.6.5
* python3 -m pip install -U pip
* yum -y --enablerepo=epel install pyOpenSSL  (on master)
* git clone https://github.com/openshift/openshift-ansible.git
* cd openshift-ansible && git fetch && git checkout release-4.0 && cd ..
*vi /etc/hosts
```bash
10.0.0.7 master console console.vipin.io
10.0.0.6 worker worker.vipin.io
```
```bash
systemctl stop docker
systemctl restart docker
systemctl enable docker
```
```bash
ssh-keygen -t rsa
ssh-copy-id root@10.0.0.7
ssh-copy-id root@10.0.0.6
```
* vi inventory.ini
* ansible-playbook -i inventory.ini openshift-ansible/playbooks/prerequisites.yml
* ansible-playbook -i inventory.ini openshift-ansible/playbooks/deploy_cluster.yml

* htpasswd -c /etc/origin/master/htpasswd admin
* oc adm policy add-cluster-role-to-user cluster-admin admin


* inventory.ini
```yaml
[OSEv3:children]
masters
nodes
etcd


[masters]
10.0.0.7 openshift_ip=10.0.0.7 openshift_schedulable=true
[etcd]
10.0.0.7 openshift_ip=10.0.0.7
[nodes]
10.0.0.7 openshift_ip=10.0.0.7 openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
10.0.0.6 openshift_ip=10.0.0.6 openshift_schedulable=true openshift_node_labels="{'region': 'primary', 'zone': 'east'}"
10.0.0.7 openshift_node_group_name="node-config-master-infra"
10.0.0.6 openshift_node_group_name="node-config-compute"

[OSEv3:vars]
debug_level=4
ansible_ssh_user=root
enable_excluders=False
enable_docker_excluder=False
openshift_enable_service_catalog=False
ansible_service_broker_install=False
openshift_disable_check=disk_availability,docker_storage,docker_image_availability,memory_availability
openshift_install_management=true

containerized=True
os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'
openshift_disable_check=disk_availability,docker_storage,memory_availability,docker_image_availability

openshift_node_kubelet_args={'pods-per-core': ['10']}

deployment_type=origin
openshift_deployment_type=origin

openshift_release=v3.11.0
openshift_pkg_version=-3.11.0
openshift_image_tag=v3.11.0
openshift_service_catalog_image_version=v3.11.0
template_service_broker_image_version=v3.11.0

osm_use_cockpit=true

openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

openshift_public_hostname=console.vipin.io
openshift_master_default_subdomain=apps.vipin.io
```
```bash
oc get pods --> registry-console pod -- crashloopbackoff
oc get dc registry-console -o yaml >a.yml
vim a.yml --> change image from docker.io/cockpit/kubernetes:latest to docker.io/timbordemann/cockpit-kubernetes:latest
oc apply -f a.yml
```
