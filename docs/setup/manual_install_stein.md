# I. Install OpenStack Stein Manual 

[1. Chuẩn bị mô hình](#1)

[2. Chuẩn bị môi trường](#2)

[2.1. Cài đặt OS](#2.1)

[2.2. Cài đặt Requirement packages](#2.2)

[2.3. Cài đặt NTPD](#2.3)

[3. Cài đặt OpenStack](#3)

[3.1. KeyStone](#3.1) 

[3.2. Glance](#3.2) 

[3.3. Nova](#3.3) 

[3.4. Neutron](#3.4)  

[3.5. Horizon](#3.5) 

[3.6. Cinder](#3.6) 

## 1.2. Mô hình cài đặt

![](../../images/noha_openstack_stein_topology.png)

## 1.3 IP Planning

![](../../images/noha_openstack_stein_ipplanning.png)

# II. Cài đặt các thành phần cần thiết <a name="2"></a>

## 2.1 Install OS <a name="2.1"></a>

> Lưu ý đến thời điểm hiện tại 2019-10-25

Cài đặt trên CentOS7 

## 2.2 Các bước cài đặt <a name="2.2"></a>

### Các bước chuẩn bị trên Controller 

- Cài đặt epel-release và các package cần thiết 
```sh
yum install -y epel-release
yum update -y && yum upgrade -y
yum install -y vim telnet wget curl byobu net-tools
```

- Thiết lập Hostname
```sh 
hostnamectl set-hostname controller
```

- Thiết lập IP
```sh 
nmcli connection show 
nmcli c modify "Wired connection 1" connection.id eth1
nmcli c modify "Wired connection 2" connection.id eth2
nmcli c modify "Wired connection 3" connection.id eth3

echo "Setup IP  eth0"
nmcli c modify eth0 ipv4.addresses 10.10.22.120/24
nmcli c modify eth0 ipv4.gateway 10.10.22.1
nmcli c modify eth0 ipv4.dns 8.8.8.8
nmcli c modify eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

echo "Setup IP eth1"
nmcli c modify eth1 ipv4.addresses 10.10.23.120/24
nmcli c modify eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

echo "Setup IP eth2"
nmcli c modify eth2 ipv4.addresses 10.10.24.120/24
nmcli c modify eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

echo "Setup IP eth3"
nmcli c modify eth3 ipv4.addresses 10.10.25.120/24
nmcli c modify eth3 ipv4.method manual
nmcli con mod eth3 connection.autoconnect yes

echo "Disable Firewall & SElinux"
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl enable network
sudo systemctl start network
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

- Disable IPv6 (Tạm thời)
```sh 
echo "
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1 " >> /etc/sysctl.conf

sysctl -p
```

- Khai báo repo và cài đặt các package cho OpenStack Stein 
```sh 
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo

yum install -y centos-release-openstack-stein python2-PyMySQL 
yum install -y python-openstackclient openstack-selinux 
yum upgrade -y
```

- Setup timezone về Ho_Chi_Minh
```sh
timedatectl set-timezone Asia/Ho_Chi_Minh
```

- Cài đặt đồng bộ thời gian 
```sh 
yum install chrony -y
systemctl enable --now chronyd
```

- Kiểm tra 
```sh 
timedatectl
```

> Kết quả 
```sh 
      Local time: T6 2019-10-25 14:02:44 +07
  Universal time: T6 2019-10-25 07:02:44 UTC
        RTC time: T6 2019-10-25 07:02:43
       Time zone: Asia/Ho_Chi_Minh (+07, +0700)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

- Bổ sung cmd_log
```sh 
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/ghichep-cmdlog/master/cmdlog.sh | bash
```

- Reboot Server 
```sh 
init 6
```

### Các bước chuẩn bị trên Compute1

- Cài đặt epel-release và các package cần thiết 
```sh
yum install -y epel-release
yum update -y && yum upgrade -y
yum install -y vim telnet wget curl byobu net-tools
```

- Thiết lập Hostname
```sh 
hostnamectl set-hostname compute1
```

- Thiết lập IP
```sh 
nmcli connection show 
nmcli c modify "Wired connection 1" connection.id eth1
nmcli c modify "Wired connection 2" connection.id eth2
nmcli c modify "Wired connection 3" connection.id eth3

echo "Setup IP  eth0"
nmcli c modify eth0 ipv4.addresses 10.10.22.121/24
nmcli c modify eth0 ipv4.gateway 10.10.22.1
nmcli c modify eth0 ipv4.dns 8.8.8.8
nmcli c modify eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

echo "Setup IP eth1"
nmcli c modify eth1 ipv4.addresses 10.10.23.121/24
nmcli c modify eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

echo "Setup IP eth2"
nmcli c modify eth2 ipv4.addresses 10.10.24.121/24
nmcli c modify eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes

echo "Setup IP eth3"
nmcli c modify eth3 ipv4.addresses 10.10.25.121/24
nmcli c modify eth3 ipv4.method manual
nmcli con mod eth3 connection.autoconnect yes

echo "Disable Firewall & SElinux"
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl enable network
sudo systemctl start network
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

- Khai báo repo và cài đặt các package cho OpenStack Stein 
```sh 
echo '[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1' >> /etc/yum.repos.d/MariaDB.repo

yum install -y epel-release
yum update -y && yum upgrade -y
yum install -y centos-release-openstack-stein python2-PyMySQL vim telnet wget curl 
yum install -y python-openstackclient openstack-selinux 
yum upgrade -y
```

- Setup timezone về Ho_Chi_Minh
```sh
timedatectl set-timezone Asia/Ho_Chi_Minh
```

- Cài đặt đồng bộ thời gian 
```sh 
yum install chrony -y
systemctl enable --now chronyd
```

- Kiểm tra 
```sh 
timedatectl
```

> Kết quả 
```sh 
      Local time: T6 2019-10-25 14:02:44 +07
  Universal time: T6 2019-10-25 07:02:44 UTC
        RTC time: T6 2019-10-25 07:02:43
       Time zone: Asia/Ho_Chi_Minh (+07, +0700)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

- Bổ sung cmd_log
```sh 
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/ghichep-cmdlog/master/cmdlog.sh | bash
```

- Reboot Server 
```sh 
init 0
```
>  ## Snapshot VM 

## 2.3 Cài đặt MySQL (Chỉ cấu hình trên Node Controller)  <a name="2.4"></a>

- Khởi động VM lên và tiến hành cài đặt 

- Cài đặt package 
```sh
yum install mariadb mariadb-server -y 
```

- Thêm config của mysql cho openstack
```sh 
cat << EOF >> /etc/my.cnf.d/openstack.cnf 
[mysqld]
bind-address = 10.10.22.120
        
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
```

- Enable và start MySQL
```sh 
systemctl enable --now mariadb.service
```

- Cài đặt passwd cho MySQL
```sh 
mysql_secure_installation <<EOF

y
passla123
passla123
y
y
y
y
EOF
```

## 2.5 Cài đặt RabbitMQ (Chỉ cấu hình trên Node Controller)  <a name="2.5"></a>

- Cài đặt package 
```sh
yum install rabbitmq-server -y
```

- Cấu hình cho rabbitmq-server
```sh
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
cd /usr/sbin/
curl -O http://localhost:15672/cli/rabbitmqadmin
chmod a+x rabbitmqadmin
cd
rabbitmqadmin list users
```

- Enable và start rabbitmq-server 
```sh 
systemctl enable --now rabbitmq-server
systemctl status rabbitmq-server
```

- Tạo user và gán quyền 
```sh
rabbitmqctl add_user openstack passla123
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmqctl set_user_tags openstack administrator
```

- Kiểm tra user vừa tạo 
```sh 
rabbitmqadmin list users
```

- Đăng nhập vào Dashboard quản trị của Rabbit-mq
```sh 
http://10.10.22.120:15672
user: openstack
password: passla123
```
![](../../images/noha_rabbitmq.png)

## 2.6 Cài đặt Memcached (Chỉ cấu hình trên Node Controller)  <a name="2.6"></a>

- Cài đặt package 
```sh 
yum install memcached python-memcached -y 
```

- Cấu hình cho memcached
```sh 
sed -i "s/-l 127.0.0.1,::1/-l 10.10.22.120/g" /etc/sysconfig/memcached
```

- Enable và start memcached
```sh 
systemctl enable --now memcached.service
```

## Cài đặt xong môi trường 

# III. Cài đặt OpenStack  <a name="3"></a>

## 3.1 Cài đặt Keystone (Service Identity) (Chỉ cấu hình trên Node Controller) <a name="3.1"></a>

### Cài đặt 

- Đăng nhập vào MySQL
```sh 
mysql
```

- Create DB và User cho Keystone
```sh
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost'  IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'passla123';
FLUSH PRIVILEGES;
exit;
```

- Cài đặt package 
```sh  
yum install -y openstack-keystone httpd mod_wsgi
```

- Backup cấu hình 
```sh 
mv /etc/keystone/keystone.{conf,conf.bk}
```

- Cấu hình cho Keystone
```sh 
cat << EOF >> /etc/keystone/keystone.conf
[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
connection = mysql+pymysql://keystone:passla123@10.10.22.120/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[federation]
[fernet_tokens]
[healthcheck]
[identity]
[identity_mapping]
[ldap]
[matchmaker_redis]
[memcache]
[oauth1]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
#driver = messagingv2
[oslo_messaging_rabbit]
#rabbit_retry_interval = 1
#rabbit_retry_backoff = 2
#amqp_durable_queues = true
#rabbit_ha_queues = true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
provider = fernet
[tokenless_auth]
[trust]
EOF
```

- Phân quyền lại config file
```sh 
chown root:keystone /etc/keystone/keystone.conf
```

- Đồng bộ database cho keystone
```sh 
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

- Thiết lập Fernet key
```sh 
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

- Thiết lập boostrap cho Keystone
```sh
keystone-manage bootstrap --bootstrap-password passla123 \
--bootstrap-admin-url http://10.10.22.120:5000/v3/ \
--bootstrap-internal-url http://10.10.22.120:5000/v3/ \
--bootstrap-public-url http://10.10.22.120:5000/v3/ \
--bootstrap-region-id RegionOne
```

- Cấu hình apache cho keystone 
```sh
sed -i 's|#ServerName www.example.com:80|ServerName 10.10.22.120|g' /etc/httpd/conf/httpd.conf 
```

- Create symlink cho keystone api
```sh 
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
ls /etc/httpd/conf.d/
```

- Start & Enable apache 
```sh 
systemctl enable --now httpd.service
systemctl restart httpd.service
systemctl status httpd.service
```

- Tạo file biến môi trường `openrc-admin` cho tài khoản quản trị
```sh 
cat << EOF >> admin-openrc
export export OS_REGION_NAME=RegionOne
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=passla123
export OS_AUTH_URL=http://10.10.22.120:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='[\u@\h \W(admin-openrc-r1)]\$ '
EOF
```

- Tạo file biến môi trường `openrc-demo` cho tài khoản  demo
```sh 
cat << EOF >> demo-openrc
export export OS_REGION_NAME=RegionOne
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=passla123
export OS_AUTH_URL=http://10.10.22.120:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='[\u@\h \W(demo-openrc-r1)]\$ '
EOF
```

### Tạo domain, projects, users, và roles

- Sử dụng biến môi trường
```sh 
source admin-openrc 
```

- Tạo Project Service
```sh
openstack project create --domain default --description "Service Project" service
```

- Tạo Project demo
```sh 
openstack project create --domain default --description "Demo Project" demo
```

- Tạo User demo và password
```sh
openstack user create --domain default --password passla123 demo
```

- Tạo roles user
```sh 
openstack role create user
```

- Thêm roles user trên Project demo 
```sh 
openstack role add --project demo --user demo user
```

- Kiểm tra xác thực password trên Project admin
```sh 
unset OS_AUTH_URL OS_PASSWORD

openstack --os-auth-url http://10.10.22.120:5000/v3 --os-project-domain-name Default \
--os-user-domain-name Default --os-project-name admin --os-username admin token issue
```

- Kiểm tra xác thực trên Project demo
```sh
openstack --os-auth-url http://10.10.22.120:5000/v3 --os-project-domain-name default \
--os-user-domain-name default --os-project-name demo --os-username demo token issue
```

- Sau khi kiểm tra xác thực xong source lại biến môi trường
```sh 
source admin-openrc 
```

- Nếu trong quá trình thao tác, xác thực token có vấn đề, get lại token mới
```sh 
openstack token issue
```

## 3.2 Cài đặt Glance (Service Images) (Chỉ cấu hình trên Node Controller) <a name="3.2"></a>

### Create DB, user và các endpoint cho Glance

- Đăng nhập vào MySQL 
```sh 
mysql
```

- Create DB và User cho Glance
```sh
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost'  IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'passla123';
FLUSH PRIVILEGES;
exit;
```

- Sử dụng biến môi trường
```sh
source admin-openrc
```

- Tạo user glance
```sh 
openstack user create --domain default --password passla123 glance
```

- Thêm roles admin cho user glance trên project service 
```sh 
openstack role add --project service --user glance admin
```

- Kiểm tra lại user glance
```sh 
openstack role list --user glance --project service
```

- Khởi tạo dịch vụ glance
```sh 
openstack service create --name glance --description "OpenStack Image" image
```

- Tạo các enpoint cho glane 
```sh 
openstack endpoint create --region RegionOne image public http://10.10.22.120:9292
openstack endpoint create --region RegionOne image internal http://10.10.22.120:9292
openstack endpoint create --region RegionOne image admin http://10.10.22.120:9292
```

###  Cài đặt cấu hình Glance

- Cài đặt package
```sh 
yum install -y openstack-glance
```

- Backup cấu hình glance-api
```sh 
mv /etc/glance/glance-api.{conf,conf.bk}
```

- Cấu hình glance-api
```sh 
cat << EOF >> /etc/glance/glance-api.conf
[DEFAULT]
bind_host = 10.10.22.120
registry_host = 10.10.22.120
[cors]
[database]
connection = mysql+pymysql://glance:passla123@10.10.22.120/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://10.10.22.120:5000
auth_url = http://10.10.22.120:5000
memcached_servers = 10.10.22.120:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = passla123
region_name = RegionOne
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
#driver = messagingv2
[oslo_messaging_rabbit]
#rabbit_ha_queues = true
#rabbit_retry_interval = 1
#rabbit_retry_backoff = 2
#amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
EOF
```

- Backup cấu hình glance-registry
```sh 
mv /etc/glance/glance-registry.{conf,conf.bk}
```

- Cấu hình glance-api
```sh 
cat << EOF >> /etc/glance/glance-registry.conf
[DEFAULT]
bind_host = 10.10.22.120
[database]
connection = mysql+pymysql://glance:passla123@10.10.22.120/glance
[keystone_authtoken]
auth_uri = http://10.10.22.120:5000
auth_url = http://10.10.22.120:5000
memcached_servers = 10.10.22.120
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = passla123
region_name = RegionOne
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
#driver = messagingv2
[oslo_messaging_rabbit]
#rabbit_ha_queues = true
#rabbit_retry_interval = 1
#rabbit_retry_backoff = 2
#amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF
```

- Phân quyền lại file cấu hình
```sh 
chown root:glance /etc/glance/*.conf
```

- Đồng bộ database cho glance 
```sh
su -s /bin/sh -c "glance-manage db_sync" glance
```

- Enable và restart Glance
```sh 
systemctl enable --now openstack-glance-api.service openstack-glance-registry.service
```

- Download image cirros
```sh 
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
```

- Upload image lên Glance
```sh 
openstack image create "cirros" --file cirros-0.3.5-x86_64-disk.img \
--disk-format qcow2 --container-format bare --public
```

- Kiểm tra images 
```sh 
openstack image list
```

## 3.3 Cài đặt Nova (Service Compute) <a name="3.3"></a> 

### 3.3.1 Cấu hình trên Node Controller

#### Create DB, user và các endpoint cho Nova trên Controller

- Đăng nhập vào MySQL
```sh 
mysql
```

- Create DB và User cho Nova
```sh
CREATE DATABASE nova;
CREATE DATABASE nova_api;
CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost'  IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost'  IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost'  IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'passla123';
FLUSH PRIVILEGES;
exit;
```

- Sử dụng biến môi trường
```sh
source admin-openrc
```

- Tạo user nova
```sh 
openstack user create --domain default --password passla123 nova
```

- Thêm role admin cho user placement trên project service
```sh 
openstack role add --project service --user nova admin
```

- Tạo dịch vụ nova
```sh 
openstack service create --name nova --description "OpenStack Compute" compute
```
    
- Tạo các endpoint cho dịch vụ compute
```sh 
openstack endpoint create --region RegionOne compute public http://10.10.22.120:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://10.10.22.120:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://10.10.22.120:8774/v2.1
```

- Tạo user placement
```sh 
openstack user create --domain default --password passla123 placement
```

- Thêm role admin cho user placement trên project service
```sh 
openstack role add --project service --user placement admin
```

- Tạo dịch vụ placement
```sh 
openstack service create --name placement --description "Placement API" placement
```

- Tạo endpoint cho placement
```sh 
openstack endpoint create --region RegionOne placement public http://10.10.22.120:8778
openstack endpoint create --region RegionOne placement internal http://10.10.22.120:8778
openstack endpoint create --region RegionOne placement admin http://10.10.22.120:8778
```

#### Cài đặt cấu hình Nova

- Cài đặt package
```sh
yum install -y openstack-nova-api openstack-nova-conductor openstack-nova-console \
openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api
```

- Backup cấu hình nova 
```sh 
mv /etc/nova/nova.{conf,conf.bk}
```

- Cấu hình cho nova 
```sh
cat << EOF >> /etc/nova/nova.conf
[DEFAULT]
my_ip = 10.10.22.120
enabled_apis = osapi_compute,metadata
use_neutron = True
osapi_compute_listen=10.10.22.120
metadata_host=10.10.22.120
metadata_listen=10.10.22.120
metadata_listen_port=8775    
firewall_driver = nova.virt.firewall.NoopFirewallDriver
allow_resize_to_same_host=True
notify_on_state_change = vm_and_task_state
transport_url = rabbit://openstack:passla123@10.10.22.120:5672
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:passla123@10.10.22.120/nova_api
[barbican]
[cache]
backend = oslo_cache.memcache_pool
enabled = true
memcache_servers = 10.10.22.120:11211
[cells]
[cinder]
os_region_name = RegionOne
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
connection = mysql+pymysql://nova:passla123@10.10.22.120/nova
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://10.10.22.120:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.22.120:5000/v3
memcached_servers = 10.10.22.120:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = passla123
region_name = RegionOne
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
region_name = RegionOne
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://10.10.22.120:5000/v3
username = placement
password = passla123
os_region_name = RegionOne
[quota]
[rdp]
[remote_debug]
[scheduler]
discover_hosts_in_cells_interval = 300
[serial_console]
[service_user]
[spice]
[upgrade_levels]
[vault]
[vendordata_dynamic_auth]
[vmware]
[vnc]
novncproxy_host=10.10.22.120
enabled = true
vncserver_listen = 10.10.22.120
vncserver_proxyclient_address = 10.10.22.120
novncproxy_base_url = http://10.10.22.120:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

- Phân quyền lại file config nova
```sh 
chown root:nova /etc/nova/nova.conf
```

- Backup cấu hình nova placement
```sh 
cp /etc/httpd/conf.d/00-nova-placement-api.{conf,conf.bk}
```

- Cấu hình virtualhost cho nova placement
```sh 
cat << 'EOF' >> /etc/httpd/conf.d/00-nova-placement-api.conf


<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
EOF
```

- Cấu hình bind cho nova placement api trên httpd 
```sh 
sed -i -e 's/VirtualHost \*/VirtualHost 10.10.22.120/g' /etc/httpd/conf.d/00-nova-placement-api.conf
sed -i -e 's/Listen 8778/Listen 10.10.22.120:8778/g' /etc/httpd/conf.d/00-nova-placement-api.conf
```

- Restart httpd 
```sh 
systemctl restart httpd 
```

- Import DB nova 
```sh 
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova 
su -s /bin/sh -c "nova-manage db sync" nova
```

> Bước import nova database bỏ qua hết các Warning `Duplicate` ...

- Check nova cell
```sh 
nova-manage cell_v2 list_cells
```

- Enable và start service nova 
```sh 
systemctl enable --now openstack-nova-api.service openstack-nova-consoleauth.service \
openstack-nova-scheduler.service openstack-nova-conductor.service \
openstack-nova-novncproxy.service
```

- Kiểm tra cài đặt lại dịch vụ 
```sh 
openstack compute service list
```

> Kết quả 
```sh 
[root@controller ~(admin-openrc-r1)]$ openstack compute service list 
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth | controller | internal | enabled | up    | 2019-10-25T08:00:49.000000 |
|  2 | nova-scheduler   | controller | internal | enabled | up    | 2019-10-25T08:00:42.000000 |
|  4 | nova-conductor   | controller | internal | enabled | up    | 2019-10-25T08:00:43.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
```

### 3.3.2 Cấu hình trên Node Compute1

- Cài đặt package
```sh
yum install -y openstack-nova-compute libvirt-client
```

- Backup cấu hình nova 
```sh 
mv /etc/nova/nova.{conf,conf.bk}
```

- Cấu hình Nova 
```sh 
cat << EOF >> /etc/nova/nova.conf 
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:passla123@10.10.22.120:5672
my_ip = 10.10.22.121
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[api_database]
[barbican]
[cache]
[cells]
[cinder]
os_region_name = RegionOne
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
[devices]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://10.10.22.120:9292
[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_url = http://10.10.22.120:5000/v3
memcached_servers = 10.10.22.120:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = passla123
region_name = RegionOne
[libvirt]
# egrep -c '(vmx|svm)' /proc/cpuinfo = 0
virt_type = qemu
#virt_type = kvm
#cpu_mode = host-passthrough
#hw_disk_discard = unmap
[matchmaker_redis]
[metrics]
[mks]
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://10.10.22.120:5000/v3
username = placement
password = passla123
os_region_name = RegionOne
[quota]
[rdp]
[remote_debug]
[scheduler]
[serial_console]
[service_user]
[spice]
[upgrade_levels]
[vault]
[vendordata_dynamic_auth]
[vmware]
[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = 10.10.22.121
novncproxy_base_url = http://10.10.22.120:6080/vnc_auto.html
[workarounds]
[wsgi]
[xenserver]
[xvp]
EOF
```

- Phân quyền lại file config
```sh 
chown root:nova /etc/nova/nova.conf
```

- Enable và start service
```sh 
systemctl enable --now libvirtd.service openstack-nova-compute.service
```

### 3.3.3 Kiểm tra trên Controller 

- Sử dụng biến môi trường
```sh
source admin-openrc
```

- Kiểm tra nova-compute 
```sh 
openstack compute service list
```

> Kết quả
```sh 
[root@controller ~(admin-openrc-r1)]$ openstack compute service list 
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth | controller | internal | enabled | up    | 2019-10-25T08:04:56.000000 |
|  2 | nova-scheduler   | controller | internal | enabled | up    | 2019-10-25T08:04:49.000000 |
|  4 | nova-conductor   | controller | internal | enabled | up    | 2019-10-25T08:04:58.000000 |
|  8 | nova-compute     | compute1   | nova     | enabled | up    | 2019-10-25T08:04:57.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
```

## 3.4 Cài đặt Neutron (Service Network) <a name="3.4"></a> 

### 3.4.1 Cấu hình trên Node Controller

#### Create DB, user và các endpoint cho Neutron trên Controller

- Đăng nhập MySQL
```sh 
mysql
```

- Create DB và User cho Nova
```sh
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'passla123';
FLUSH PRIVILEGES;
exit;
```

- Sử dụng biến môi trường
```sh
source admin-openrc
```
    
- Tạo user neutron
```sh 
openstack user create --domain default --password passla123 neutron
```
	
- Gán role admin cho user neutron 
```sh 
openstack role add --project service --user neutron admin
```
	
- Khởi tạo dịch vụ neutron
```sh 
openstack service create --name neutron --description "OpenStack Networking" network
```

- Tạo các endpoint cho neutron 
```sh 
openstack endpoint create --region RegionOne network public http://10.10.22.120:9696
openstack endpoint create --region RegionOne network internal http://10.10.22.120:9696
openstack endpoint create --region RegionOne network admin http://10.10.22.120:9696
```

#### Cài đặt cấu hình neutron

- Cài đặt package
```sh 
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

Lưu ý: Mô hình này sử dụng mô hình mạng provider (flat) sử dụng linuxbridge
DHCP agent và metadata agent được chạy trên node compute

- Backup cấu hình neutron 
```sh
mv /etc/neutron/neutron.{conf,conf.bk}
```

- Cấu hình neutron 
```sh 
cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
bind_host = 10.10.22.120
core_plugin = ml2
service_plugins = router
transport_url = rabbit://openstack:passla123@10.10.22.120
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
allow_overlapping_ips = True
dhcp_agents_per_network = 2
[agent]
[cors]
[database]
connection = mysql+pymysql://neutron:passla123@10.10.22.120/neutron
[keystone_authtoken]
auth_uri = http://10.10.22.120:5000
auth_url = http://10.10.22.120:5000
memcached_servers = 10.10.22.120:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = passla123
region_name = RegionOne
[matchmaker_redis]
[nova]
auth_url = http://10.10.22.120:5000
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = passla123
region_name = RegionOne
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues = true
rabbit_ha_queues = true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[quotas]
[ssl]
EOF
```

- Phân quyền lại file config 
```sh 
chown root:neutron /etc/neutron/neutron.conf
```

Cấu hình ml2_config

- Backup config ml2_config
```sh 
mv /etc/neutron/plugins/ml2/ml2_conf.{ini,ini.bk}
```

- Cấu hình ml2_config
```sh 
cat << EOF >> /etc/neutron/plugins/ml2/ml2_conf.ini
[DEFAULT]
[l2pop]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
# mechanism_drivers = linuxbridge,l2population
mechanism_drivers = linuxbridge
extension_drivers = port_security
[ml2_type_flat]
flat_networks = provider
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
# network_vlan_ranges = provider
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = true
EOF
```

- Phân quyền lại file ml2_config
```sh 
chown root:neutron /etc/neutron/plugins/ml2/ml2_conf.ini
```

Cấu hình linuxbridge_agent

- Backup lại config linuxbridge_agent
```sh 
mv /etc/neutron/plugins/ml2/linuxbridge_agent.{ini,init.bk}
```

- Cấu hình cho linuxbridge_agent
```sh 
cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eth1
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
# enable_vxlan = true
## network dataVM
local_ip = 10.10.24.120
# l2_population = true
EOF
```

- Phân quyền lại file linuxbridge_agent
```sh 
chown root:neutron /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

Cấu hình trên file l3_agent

- Backup cấu hình l3_agent
```sh 
mv /etc/neutron/l3_agent.{ini,ini.bk}
```

- Cấu hình l3_agent 
```sh
cat << EOF >> /etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
[agent]
[ovs]
EOF
```

- Phân quyền lại file config 
```sh 
chown root:neutron /etc/neutron/l3_agent.ini
```

Bổ sung cấu hình phép nova service trên controller sử dụng networking service 

- Chỉnh sửa bổ sung cấu hình block `[neutron]` trong file `/etc/nova/nova.conf`
```sh
echo "[neutron]
url = http://10.10.22.120:9696
auth_url = http://10.10.22.120:5000
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = passla123
service_metadata_proxy = true
metadata_proxy_shared_secret = passla123
region_name = RegionOne " >> /etc/nova/nova.conf
```


Các Networking service initialization script yêu cầu symbolic link `/etc/neutron/plugin.ini` tới ML2 plug-in config file `/etc/neutron/plugins/ml2/ml2_conf.ini`

- Khởi tạo symlink cho ml2_config 
```sh 
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

- Đồng bộ database
```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

- Khởi động lại các dịch vụ Nova:
```sh 
systemctl restart openstack-nova-api.service openstack-nova-scheduler.service \
openstack-nova-consoleauth.service openstack-nova-conductor.service \
openstack-nova-novncproxy.service
```

- Enable và restart Neutron service 
```
systemctl enable --now neutron-server.service neutron-linuxbridge-agent.service \
neutron-l3-agent.service
```

### 3.4.2 Cấu hình trên Node Compute1 

- Cài đặt các package 
```sh 
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

Cấu hình neutron 

- Backup neutron config 
```sh 
mv /etc/neutron/neutron.{conf,conf.bk}
```

- Cấu hình neutron 
```sh 
cat << EOF >> /etc/neutron/neutron.conf
[DEFAULT]
transport_url = rabbit://openstack:passla123@10.10.22.120:5672
auth_strategy = keystone
[agent]
[cors]
[database]
[keystone_authtoken]
auth_uri = http://10.10.22.120:5000
auth_url = http://10.10.22.120:5000
memcached_servers = 10.10.22.120:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = passla123
region_name = RegionOne
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
#driver = messagingv2
[oslo_messaging_rabbit]
rabbit_ha_queues = true
rabbit_retry_interval = 1
rabbit_retry_backoff = 2
amqp_durable_queues= true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[quotas]
[ssl]
EOF
```

- Phân quyền lại file nova config
```sh 
chown root:neutron /etc/neutron/neutron.conf
```

Cấu hình linuxbridge_agent

> Linux bridge agent xây dựng layer-2 (bridging và switching) virtual networking infrastructure cho instances và xử lý các security group.

- Backup linuxbridge_agent config 
```sh 
mv /etc/neutron/plugins/ml2/linuxbridge_agent.{ini,ini.bk}
```

- Config linuxbridge_agent
```sh 
cat << EOF >> /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eth1
[network_log]
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
# enable_vxlan = true
## network dataVM
local_ip = 10.10.24.121
# l2_population = true
EOF
```

- Phân quyền lại cho file linuxbridge_agent config 
```sh 
chown root:neutron /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

Cấu hình DHCP agent

- Backup cấu hình dhcp_agent 
```sh 
mv /etc/neutron/dhcp_agent.{ini,ini.bk}
```

- Cấu hình dhcp_agent
```sh 
cat << EOF >> /etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
force_metadata = True
[agent]
[ovs]
EOF
```

- Phân quyền lại file config 
```sh 
chown root:neutron /etc/neutron/dhcp_agent.ini
```

Cấu hình metadata_agent

- Backup file cấu hình 
```sh 
mv /etc/neutron/metadata_agent.{ini,ini.bk}
```

- Chỉnh sửa config 
```sh 
cat << EOF >> /etc/neutron/metadata_agent.ini
[DEFAULT]
# IP mngt controller 
nova_metadata_host = 10.10.22.120
metadata_proxy_shared_secret = passla123
[agent]
[cache]
EOF
```

- Phân quyền lại file config 
```sh 
chown root:neutron /etc/neutron/metadata_agent.ini
```

Để nova services trên node compute có thể sử dụng networking service thì chúng ta bổ sung thêm cấu hình cho nova.conf như sau 

- Bổ sung cấu hình phần `[neutron]` trong `/etc/nova/nova.conf`
```sh 
echo "[neutron]
url = http://10.10.22.120:9696
auth_url = http://10.10.22.120:5000
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = passla123
region_name = RegionOne " >> /etc/nova/nova.conf
```

- Restart lại Compute service
```sh 
systemctl restart openstack-nova-compute.service libvirtd.service 
```

- Enable và start linuxbridge_agent dhcp_agent metadata_agent
```sh 
systemctl enable --now neutron-linuxbridge-agent.service \
neutron-dhcp-agent.service neutron-metadata-agent.service
```

###  3.4.3 Quay lại node Controller kiểm tra

Kiểm tra 
```sh 
source admin-openrc
openstack network agent list
```

> Kết quả
```sh 
[root@controller ~(admin-openrc-r1)]$ openstack network agent list 
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 80da032e-8c70-401d-8b4e-a6f2d8d58f17 | Metadata agent     | compute1   | None              | :-)   | UP    | neutron-metadata-agent    |
| 99a61a9c-f30a-4bde-be72-26f56ee25f91 | Linux bridge agent | compute1   | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 9d70e5e2-5e0f-4c67-9850-dca17be3e5d0 | L3 agent           | controller | nova              | :-)   | UP    | neutron-l3-agent          |
| ea1d1198-90c3-4fac-9d7a-1f42c8f4bf2e | DHCP agent         | compute1   | nova              | :-)   | UP    | neutron-dhcp-agent        |
| ea6adbbf-2786-4d64-a345-d8a3469df4d4 | Linux bridge agent | controller | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```

## 3.5 Cài đặt Horizon (Service Dashboard) (Chỉ cấu hình trên Node Controller) <a name="3.5"></a>

### Cài đặt cấu hình 

- Cài đặt packages
```sh 
yum install -y openstack-dashboard
```

- Tạo file direct
```sh 
filehtml=/var/www/html/index.html
touch $filehtml
cat << EOF >> $filehtml
<html>
<head>
<META HTTP-EQUIV="Refresh" Content="0.5; URL=http://10.10.22.120/dashboard">
</head>
<body>
<center> <h1>Redirecting to OpenStack Dashboard</h1> </center>
</body>
</html>
EOF
```
	
- Backup file cấu hình 
```sh 
mv  /etc/openstack-dashboard/{local_settings,local_settings.bk}
```

- Cập nhật cấu hình 
```sh
curl --url https://raw.githubusercontent.com/uncelvel/tutorial-openstack/master/docs/template/local_settings --output /etc/openstack-dashboard/local_settings

new_secret=$(cat /dev/urandom | tr -dc 'a-z0-9' | fold -w 20 | head -n 1)
sed -Ei "s|00000000|$new_secret|g" /etc/openstack-dashboard/local_settings

sed -Ei 's|0.0.0.0|10.10.22.120|g' /etc/openstack-dashboard/local_settings

chmod 640 /etc/openstack-dashboard/local_settings
chown root:apache /etc/openstack-dashboard/local_settings
```
	
- Thêm config httpd cho dashboard
```sh 
echo "WSGIApplicationGroup %{GLOBAL}" >> /etc/httpd/conf.d/openstack-dashboard.conf
```

- Restart service httpd và memcached
```sh 
systemctl restart httpd.service memcached.service
```	

### Hoàn tất cài đặt truy cập Dashboard 

```sh 
http://10.10.22.120
user: admin
password: passla123
```

![](../../images/noha_dashboard.png)

### Tạo network
Truy cập `Admin` --> `Network` --> `Networks` Chọn `Create Network`
![](../../images/noha_createnetwork1.png)
![](../../images/noha_createnetwork2.png)
![](../../images/noha_createnetwork3.png)
![](../../images/noha_createnetwork4.png)


### Tạo Flavor
Truy cập `Admin` --> `Compute` --> `Flavors` Chọn `Create Flavor`
![](../../images/noha_createflavor01.png)

### Boot thử 1 máy ảo 
Truy cập `Projects` --> `Compute` --> `Instance` Chọn `Launch Instance`
![](../../images/noha_createvm01.png)
![](../../images/noha_createvm02.png)
![](../../images/noha_createvm03.png)
![](../../images/noha_createvm04.png)

> VM tạo thành công nhưng chưa ping được ra ngoài, SSH ngoài vào ko OK 

### Mở Security Group 
Truy cập `Projects` --> `Network` --> `Security Groups` Chọn `Manage Rules`
![](../../images/noha_sc01.png)

Chọn `Add rules`
![](../../images/noha_sc02.png)

> Kiểm tra lại VM ok có thể ping 


## 3.6 Cài đặt Cinder (Service Storage) với backend là LVM (Chỉ cấu hình trên Node Controller) <a name="3.6"></a>

Chuẩn bị: Add thêm 1 ổ `/dev/vdb` tối thiểu 100G vào Node Controller

- Kiểm tra ổ 
```sh
[root@controller ~(admin-openrc)]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                      11:0    1 1024M  0 rom  
sda                     252:0    0   50G  0 disk 
├─sda1                  252:1    0    4G  0 part /boot
├─sda2                  252:2    0   38G  0 part 
│ └─VolGroup00-LogVol01 253:0    0   38G  0 lvm  /
└─vda3                  252:3    0    8G  0 part [SWAP]
vdb                     252:16   0  100G  0 disk 
[root@controller ~(admin-openrc)]$ 
```

- Đăng nhập MySQL
```sh 
mysql -u root -p
```

- Create DB và User cho Cinder
```sh 
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'passla123';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'passla123';
exit
```

- Sử dụng biến môi trường
```sh
source admin-openrc
```

- Tạo cinder user:
```sh 
openstack user create --domain default --password passla123 cinder
```

- Tạo admin role cho cinder user
```
openstack role add --project service --user cinder admin
```

- Tạo đối tượng cinderv2 và cinderv3 service:
```sh 
openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3
```

- Tạo Block Storage service API endpoints:
```sh 
openstack endpoint create --region RegionOne volumev2 \
public http://10.10.22.120:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 \
internal http://10.10.22.120:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 \
admin http://10.10.22.120:8776/v2/%\(project_id\)s

openstack endpoint create --region RegionOne volumev3 \
public http://10.10.22.120:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 \
internal http://10.10.22.120:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 \
admin http://10.10.22.120:8776/v3/%\(project_id\)s
```

- Cài đặt packages
```sh 
yum install -y lvm2 device-mapper-persistent-data openstack-cinder targetcli
```

- Enable và start LVM metadata service
```
systemctl enable lvm2-lvmetad.service
systemctl start lvm2-lvmetad.service
```

- Chỉnh sửa file `/etc/lvm/lvm.conf` bổ sung filter cho `vdb`
```sh 
    # Line 142
    filter = [ "a/vdb/", "r/.*/"]
```

- Tạo LVM volume /dev/vdb:
```
pvcreate /dev/vdb
vgcreate cinder-volumes /dev/vdb
```

Cấu hình Cinder 

- Backup cấu hình cinder 
```sh 
mv /etc/cinder/cinder.{conf,conf.bk}
```

- Cấu hình cho Cinder 
```sh 
cat << EOF >> /etc/cinder/cinder.conf
[DEFAULT]
transport_url = rabbit://openstack:passla123@10.10.22.120
auth_strategy = keystone
my_ip = 10.10.22.120
enabled_backends = lvm
glance_api_servers = http://10.10.22.120:9292
#rpc_backend = rabbit
#control_exchange = cinder
[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = lioadm
[backend]
[backend_defaults]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[database]
connection = mysql+pymysql://cinder:passla123@10.10.22.120/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://10.10.22.120:5000
auth_url = http://10.10.22.120:5000
memcached_servers = 10.10.22.120:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = passla123
region_name = RegionOne
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
#driver = messagingv2
[oslo_messaging_rabbit]
#rabbit_retry_interval = 1
#rabbit_retry_backoff = 2
#amqp_durable_queues = true
#rabbit_ha_queues = true
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[oslo_reports]
[oslo_versionedobjects]
[profiler]
[service_user]
[ssl]
[vault]
EOF
```

- Phân quyền lại cho file cấu hình 
```sh 
chown root:cinder /etc/cinder/cinder.conf
```

- Đồng bộ database cho Cinder
```sh 
su -s /bin/sh -c "cinder-manage db sync" cinder
```

- Restart lại nova service 
```
systemctl restart openstack-nova-api.service
```

Enable và start Block Storage services
```sh
systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service openstack-cinder-volume.service 
systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service openstack-cinder-volume.service 
```

Kiểm tra 
```sh 
[root@controller ~(admin-openrc)]$ cinder service-list 
+------------------+-----------------+------+---------+-------+----------------------------+-----------------+
| Binary           | Host            | Zone | Status  | State | Updated_at                 | Disabled Reason |
+------------------+-----------------+------+---------+-------+----------------------------+-----------------+
| cinder-scheduler | controller      | nova | enabled | up    | 2019-05-06T07:20:02.000000 | -               |
| cinder-volume    | controller@lvm  | nova | enabled | up    | 2019-05-02T07:42:52.000000 | -               |
+------------------+-----------------+------+---------+-------+----------------------------+-----------------+
```
