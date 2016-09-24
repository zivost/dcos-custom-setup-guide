
# DC/OS Private Cloud Setup

Minimum Server Requirements [details on dcos.io](https://dcos.io/docs/1.8/administration/installing/custom/system-requirements/):
 - 1 Bootstap Node
 - 1 Master Node (2 Core, 3.5 GiG)
 - 1 Agent Node (recommended 3 nodes) (2 Core, 3.5 GiG)

## 1. Install Prerequisites on all nodes
- Recommended OS: CentOS 7.2
- username should be same on all nodes
### Part 1.1
```bash
sudo yum upgrade --assumeyes --tolerant
sudo yum update --assumeyes
```
```bash
sudo tee /etc/modules-load.d/overlay.conf <<-'EOF'
overlay
EOF
reboot 
```
### Part 1.2
```bash
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d && sudo tee /etc/systemd/system/docker.service.d/override.conf <<- EOF
[Service]
ExecStart=
ExecStart=/usr/bin/docker daemon --storage-driver=overlay -H fd://
EOF
```
```bash
sudo yum install -y docker-engine-1.11.2
sudo systemctl start docker
sudo systemctl enable docker
```
### Part 1.3
```bash
sudo groupadd nogroup
sudo yum install -y ipset unzip wget git curl xz
sudo setenforce 0
mkdir /tmp/dcos
sudo kill `sudo lsof -t -i:53`
```
```bash
sudo vi /etc/sysconfig/selinux
## Change SELINUX to `permissive`
```
```bash
## to verify installation
sudo docker ps
```

## 2. Setup Bootstrap Node

```bash
cd /tmp/dcos
sudo mkdir -p genconf
```
```bash
sudo vi genconf/ip-detect
# see repository for content
```
```bash
sudo vi genconf/config.yaml
# see repository for content
```
```bash
ssh-keygen
# enter for all
# no password/passphrase
```
```bash
sudo cp ~/.ssh/id_rsa genconf/ssh_key
```

### Part 2.2
`loop the below process for each node (from bootstrap node){`
```
ssh-copy-id <username>@<node-private-ip>
sudo tee /etc/environment <<-'EOF'
LC_ALL="en_US.utf8"
EOF
exit
```
`}end of loop`

### Part 2.3
```bash
cd /tmp/dcos
curl -O https://downloads.dcos.io/dcos/stable/dcos_generate_config.sh
sudo bash dcos_generate_config.sh
sudo docker run -d -p 9000:80 -v $PWD/genconf/serve:/usr/share/nginx/html:ro nginx
```

## 3. Setup Master Nodes

`loop the below process for each master node{`
```bash
cd /tmp/dcos
curl -O http://<bootstrap-public-ip>:9000/dcos_install.sh
sudo bash dcos_install.sh master
```
`}end of loop`

## 4. Setup Private Agent Nodes

`loop the below process for each private agent node{`
```bash
cd /tmp/dcos
curl -O http://<bootstrap-public-ip>:9000/dcos_install.sh
sudo bash dcos_install.sh slave
```
`}end of loop`

## 5. Setup Public Agent Nodes

`loop the below process for each public agent node{`
```bash
cd /tmp/dcos
curl -O http://<bootstrap-public-ip>:9000/dcos_install.sh
sudo bash dcos_install.sh slave_public
```
`}end of loop`

## Notes:
 - `sudo bash dcos_install.sh <role>`  can take some time.
 - `http://<master-public-ip>` will take you to DC/OS Panel.
