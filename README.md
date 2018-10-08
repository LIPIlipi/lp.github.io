# <center>openstack-queens</center>
```python
##### 安装实验环境

创建两台虚拟机：
controller119：192.168.1.119
compuer159：192.168.1.159 

修改主机名
controller119   conputer159  都需要做
hostnamectl set-hostname controller119      #修改主机名为controller119
hostnamectl                                 #查看此时的主机名
logout                                      #退出重新登录即可生

配置网络
vim /etc/sysconfig/network-scripts/ifcfg-ens160           #配置文件controller19
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.1.119
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=114.114.114.114
NAME=ens160
DEVICE=ens160
ONBOOT=yes
systemctl restart network               #配置文件修改后重启网卡
ip a                                    #查看是否生效   ping www.baidu.com

vim /etc/sysconfig/network-scripts/ifcfg-ens160          #配置文件computer159
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.1.159
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=114.114.114.114
NAME=ens160
DEVICE=ens160
ONBOOT=yes

systemctl restart network           #配置文件修改后重启网卡
ip a                                #查看是否生效      ping www.baidu.com

##### 关闭防火墙
controller119   conputer159  都需要做
systemctl disable firewalld     #关闭开机自启
systemctl status firewalld      #查看防火墙状态
systemctl stop firewalld         #关闭防火墙

##### 修改selinux
controller119   conputer159  都需要做
vim /etc/selinux/config 打开修改配置文件
vim /etc/sysconfig/selinux
SELINUX=enforing      改为   SELINUX=disabled

##### 域名解析
controller119   conputer159  都需要做
[root@controller119 ~]# vim /etc/hosts
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.119  controller119
192.168.1.159  computer159
[root@controller119 ~]# ping controller119      # 验证域名解析
[root@controller119 ~]# ping computer159

##### 时钟同步
yum install chronyd
systemctl enable chronyd
systemctl start chronyd
systemctl status chronyd
vim /etc/chrony.conf                         #配置文件
# Please consider joiningthe pool (http://www.pool.ntp.org/join.html)
server ntp5.aliyun.com iburst
# Allow NTP client access from local network
allow 192.168.0.0/16
timedatectl set-timezone Asia/Shanghai

systemctl restart chronyd
timedatectl status

computer159的配置文件
yum install chronyd
systemctl enable chronyd
systemctl start chronyd
systemctl status chronyd
vim /etc/chrony.conf       #配置文件
# Please consider joiningthe pool (http://www.pool.ntp.org/join.html)
server 192.168.1.119 iburst

timedatectl set-timezone Asia/Shanghai
systemctl restart chronyd
timedatectl status
cd /etc/yum.repos.d/
mkdir bk
mv CentOS-* bk
mv ./bk/CentOS-Base.repo ./
修改软件源：
此项操作皆是在controller119操作的，新添加的cloud-queeus 和 epel两个清华大学的mirrors:
vim /etc/yum.repo.d/CentOS-Base.repo          #编辑配置文件
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
remarked out baseurl= line instead.
                             

[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
1baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates 
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
ame=CentOS-$releasever - Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 
[cloud-Queens]
name=Queens
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/7/cloud/x86_64/openstack-queens/
gpgcheck=0
 
[epel]
name=epel
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/7/x86_64
gpgcheck=0

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

yum clean all
yum repolist
yum update       #将更新后重新出现的其它yum源,删除其他源

#### 安装openstack客户端(controller119)
yum install python-openstackclient
yum install openstack-selinux

#### 数据库SQL database (controller119)
yum install mariadb mariadb-server python2-PyMySQL
vim /etc/my.cnf.d/openstack.cnf                                    #编辑配置文件
[mysqld]
#bind-address = 192.168.1.119
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
systemctl enable mariadb.service
systemctl start mariadb.service
mysql_secure_installation                                        #设置密码 000000

#### 消息服务器 message queue(controller119)
yum install rabbitmq-server
systemctl enable rabbitmq-server.service                          #开机自启
systemctl start rabbitmq-server.service                           #开启服务
rabbitmqctl add_user openstack 000000                             #创建用户openstack 用户密码000000
Creating user "openstack" ...
# rabbitmqctl set_permissions openstack ".*" ".*" ".*" 
#分配权限等级Setting permissions for user "openstack" in vhost "/" ...
验证：
rabbitmq list_uesrs                                                #查看用户列表
rabbitmq list_permissions                                          #查看权限列表
rabbitmq-plugins enable rabbitmq_management                        #开启组件
curl 127.0.0.1:15672                                               #web验证

#### Memcached 服务 (controller119)
yum install memcached python-memcached
vim /etc/sysconfig/memcached 
OPTIONS="-l 127.0.0.1,::1,controller119"

systemctl enable memcached.service
systemctl start memcached.service
 
#### Etcd服务(controller119)
yum install etcd
vim /etc/etcd/etcd.conf#[Member]
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://192.168.1.119:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.119:2379,http:127.0.0.1:2379"          
ETCD_NAME="controller119"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.119:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.119:2379"
ETCD_INITIAL_CLUSTER="controller119=http://192.168.1.119:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"

systemctl enable etcd
systemctl start etcd


安装服务：（注意再合适地方改变主机名和密码）

#### Keystone服务（controller19）
mysql -u root -p000000
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
 IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
 IDENTIFIED BY '000000';

安装下列包：
yum install openstack-keystone httpd mod_wsgi
vim  /etc/keystone/keystone.conf
[database]
connection = mysql+pymysql://keystone:000000@controller119/keystone
[token]
provider = fernet
su -s /bin/sh -c "keystone-manage db_sync" keystone
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://controller119:35357/v3/ \
  --bootstrap-internal-url http://controller119:5000/v3/ \
  --bootstrap-public-url http://controller119:5000/v3/ \
  --bootstrap-region-id RegionOne

vim  /etc/httpd/conf/httpd.conf
ServerName controller119

ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
systemctl enable httpd.service
systemctl start httpd.service
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller119:35357/v3
export OS_IDENTITY_API_VERSION=3

验证：
openstack domain create --description "An Example Domain" example
openstack project create --domain default \ --description "Service Project" service
openstack project create --domain default \ --description "Demo Project" demo
openstack user create --domain default \ --password-prompt demo
openstack role create user
以上验证会出现表格：
openstack role add --project demo --user demo user

unset OS_AUTH_URL OS_PASSWORD
openstack --os-auth-url http://controller119:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
openstack --os-auth-url http://controller119:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name demo --os-username demo token isexport OS_USER_DOMAIN_NAME=Default

export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller119:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

vim demo-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=000000
export OS_AUTH_URL=http://controller119:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

 . admin-openrc
openstack token issue

#### glance服务
mysql -u root -p000000
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY '000000';

. admin-openrc
openstack user create --domain default --password-prompt glance
openstack role add --project service --user glance admin
openstack service create --name glance \
  --description "OpenStack Image" image
openstack endpoint create --region RegionOne \
  image public http://controller119:9292
openstack endpoint create --region RegionOne \
  image internal http://controller119:9292
$ openstack endpoint create --region RegionOne \
  image admin http://controller119:9292

yum install openstack-glance
vim  /etc/glance/glance-api.conf
[database]
connection = mysql+pymysql://glance:000000@controller119/glance
[keystone_authtoken]
auth_uri = http://controller119:5000
auth_url = http://controller119:5000
memcached_servers = controller119:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 000000
[paste_deploy]
flavor = keystone
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/

vim /etc/glance/glance-registry.conf
[database]
connection = mysql+pymysql://glance:000000@controller/glance
[keystone_authtoken]
auth_uri = http://controller119:5000
auth_url = http://controller119:5000
memcached_servers = controller119:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 000000
[paste_deploy]
flavor = keystone

su -s /bin/sh -c "glance-manage db_sync" glance
systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service
systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
  
验证：
. admin-openrc
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public

#### nova服务： 
mysql -u root -p000000
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
 IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
 IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
 IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
 IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
 IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
 IDENTIFIED BY '000000';
  
. admin-openrc
openstack user create --domain default --password-prompt nova
openstack role add --project service --user nova admin
openstack service create --name nova \
 --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne \
 compute public http://controller119:8774/v2.1
openstack endpoint create --region RegionOne \
 compute internal http://controller119:8774/v2.1
openstack endpoint create --region RegionOne \
 compute admin http://controller119:8774/v2.1
openstack user create --domain default --password-prompt placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
openstack endpoint create --region RegionOne placement public http://controller119:8778
openstack endpoint create --region RegionOne placement internal http://controller119:8778
openstack endpoint create --region RegionOne placement admin http://controller119:8778
   
yum install openstack-nova-api openstack-nova-conductor \
 openstack-nova-console openstack-nova-novncproxy \
 openstack-nova-scheduler openstack-nova-placement-api
  
vim /etc/nova/nova.conf
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:000000@controller119
my_ip = 192.168.1.119
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api_database]
connection = mysql+pymysql://nova:000000@controller119/nova_api
[database]
connection = mysql+pymysql://nova:000000@controller119/nova
[api]
auth_strategy = keystone
[keystone_authtoken]
auth_uri = http://controller119:5000
auth_url = http://controller119:35357
memcached_servers = controller119:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 000000
[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip
[glance]
api_servers = http://controller119:9292
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller119:35357/v3
username = placement
password = 000000

vim /etc/httpd/conf.d/00-nova-placement-api.conf
<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
 
systemctl restart httpd 
su -s /bin/sh -c "nova-manage api_db sync" nova 
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova 
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
nova-manage cell_v2 list_cells

systemctl enable openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl start openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
  
compute119节点：
配置新源加源文件如下所示：
[virt]
name=kvm-common
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/7/virt/x86_64/kvm-common/
enabled=1
gpgcheck=0

yum makecache  #更新缓存
yum install openstack-nova-compute
vim /etc/nova/nova.conf
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:000000@controller119
my_ip = 192.168.1.159
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[keystone_authtoken]
auth_uri = http://controller119:5000
auth_url = http://controller119:35357
memcached_servers = controller119:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 000000
[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://controller119:6080/vnc_auto.html
[glance]
api_servers = http://controller119:9292
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller119:35357/v3
username = placement
password = 000000

egrep -c '(vmx|svm)' /proc/cpuinfo
vim /etc/nova/nova.conf
[libvirt]
virt_type = qemu

systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service

在控制controller19节点验证
. admin-openrc
openstack compute service list --service nova-compute
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

vim /etc/nova/nova.conf
[scheduler]
discover_hosts_in_cells_interval = 300

#### neutron服务  controller节点
mysql -u root -p000000
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY '000000';

. admin-openrc
openstack user create --domain default --password-prompt neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron \
 --description "OpenStack Networking" network
openstack endpoint create --region RegionOne \
 network public http://controller119:9696
openstack endpoint create --region RegionOne \
 network internal http://controller119:9696
openstack endpoint create --region RegionOne \
 network admin http://controller119:9696

  
yum install openstack-neutron openstack-neutron-ml2 \
  openstack-neutron-linuxbridge ebtables
  
vim /etc/neutron/neutron.conf
[database]
connection = mysql+pymysql://neutron:000000@controller119/neutron
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:000000@controller119
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[keystone_authtoken]
auth_uri = http://controller119:5000
auth_url = http://controller119:35357
memcached_servers = controller119:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 000000
[nova]
auth_url = http://controller119:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = 000000
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp

vim /etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
flat_networks = provider
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = true

vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[linux_bridge]
physical_interface_mappings = provider:ens160
[vxlan]
enable_vxlan = true
local_ip = 192.168.1.119
l2_population = true
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-ip6tables

vim /etc/neutron/l3_agent.ini
interface_driver = linuxbridge

vim /etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true

vim /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = controller19
metadata_proxy_shared_secret = 000000

vim /etc/nova/nova.conf
[neutron]
url = http://controller119:9696
auth_url = http://controller119:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 000000
service_metadata_proxy = true
metadata_proxy_shared_secret = 000000

ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
systemctl restart openstack-nova-api.service  
systemctl enable neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
systemctl start neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
systemctl enable neutron-l3-agent.service
systemctl start neutron-l3-agent.service  


computer119计算节点：
yum install openstack-neutron-linuxbridge ebtables ipset
vim  /etc/neutron/neutron.conf
[DEFAULT]
transport_url = rabbit://openstack:000000@controller119
auth_strategy = keystone
[keystone_authtoken]

auth_uri = http://controller119:5000
auth_url = http://controller119:35357
memcached_servers = controller119:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 000000
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
 
vim  /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[linux_bridge]
physical_interface_mappings = provider:ens160
[vxlan]
enable_vxlan = true
local_ip = 192.168.1.159
l2_population = true
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-ip6tables
vim  /etc/nova/nova.conf
[neutron]
url = http://controller119:9696
auth_url = http://controller119:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 000000

systemctl restart openstack-nova-compute.service
systemctl enable neutron-linuxbridge-agent.service
systemctl start neutron-linuxbridge-agent.service
验证：
openstack network agent list    # 出现五条，其中一条为计算节点


yum install openstack-dashboard
vim /etc/openstack-dashboard/local_settings
OPENSTACK_HOST = "controller119"
ALLOWED_HOSTS = ['*',]
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller119:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
OPENSTACK_NEUTRON_NETWORK = {
    ...
    'enable_router': False,
    'enable_quotas': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': False,
    'enable_firewall': False,
    'enable_vpn': False,
    'enable_fip_topology_check': False,
}
TIME_ZONE = "Asia/Shanghai"
注释两行 370行左右，一个vnc 一个物理网卡
验证：
http://controller/dashboard

域名：default
用户名：admin  或者 demo 
密码：000000
```
