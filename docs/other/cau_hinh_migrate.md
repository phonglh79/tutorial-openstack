# Cấu hình Cold-migrate (SCP - SSH connection)

Bước 1: Trên cả 2 node Compute: Bổ sung `/etc/hosts`
```sh 
echo "
10.10.22.81 Compute1
10.10.22.85 Compute2" >> /etc/hosts
```

Bước 2: Trên cả 2 node Compute: Tạo user `nova` và đặt passwd
```sh 
usermod -s /bin/bash nova
passwd nova
```
> Đặt passwd bất kỳ (Cần lưu lại để cấu hình các bước tiếp theo)

Bước 3: Trên Compute1 tiến hành tạo ssh-keygen cho nova user 
```sh 
su nova 
cd 
cat /dev/zero | ssh-keygen -q -N ""
```

Bước 4: Trên Compute1: Tạo file authorized và config 
```sh 
su nova 
cd 
echo 'StrictHostKeyChecking no' > /var/lib/nova/.ssh/config
cat .ssh/id_rsa.pub >> /var/lib/nova/.ssh/authorized_keys
exit 
chmod 644 /var/lib/nova/.ssh/authorized_keys
```

Bước 5: Trên Compute1: Kiểm tra các file trong thư mục `.ssh`
```sh 
su nova 
cd 
ls -lah .ssh/
```

> Kết quả 
```sh 
bash-4.2$ ls -lah .ssh/
total 20K
drwx------ 2 nova nova   94 15:18 10 Th09 .
drwxr-xr-x 9 nova nova  131 15:17 10 Th09 ..
-rw-r--r-- 1 nova nova  400 15:17 10 Th09 authorized_keys
-rw-r--r-- 1 nova nova   25 15:17 10 Th09 config
-rw------- 1 nova nova 1,7K 15:17 10 Th09 id_rsa
-rw-r--r-- 1 nova nova  400 15:17 10 Th09 id_rsa.pub
bash-4.2$ 
```


Bước 6: Trên Compute2: Tạo folder `/var/lib/nova/.ssh/`
```sh 
su nova 
cd
mkdir -p /var/lib/nova/.ssh/
```

Bước 7: Trên Compute1: Copy toàn bộ trong thư mục .ssh qua Compute2
```sh 
su nova 
cd 
scp Compute2:.ssh/authorized_keys .ssh/
```
> Nhập passwd của nova user (Đã tạo ở bước 2)

Bước 8: Đứng trên 2 Compute truy cập nova user và tiến hành ssh qua lại lẫn nhau xem có cần passwd không 

- Trên Compute1
```sh 
su nova 
cd 
ssh Compute2
```

- Trên Compute2
```sh 
su nova 
cd 
ssh Compute1
```

> SSH không cần passwd là thành công chức năng cold-migrate đã sẵn sàng.  

## Kiểm tra việc migrate

- Thực hiện login và controller và source admin environment
```sh
ssh root@192.168.70.11
source admin-openrc
```

- Migrate máy ảo. 
```sh 
nova migrate --poll $VM_ID
```
> Thay $VM_ID với id của máy ảo. 

Sau khi máy ảo chuyển trạng thái Finished thực hiện confirm.  
```sh
nova resize-confirm $VM_ID
```

- Start máy ảo
```sh 
nova start $VM_ID 
```

- Login vào máy ảo vào kiểm tra

# Cấu hình Live migrate (TCP connection)
Sửa thông tin trong libvirt của VM mơi
```sh 
cp /etc/libvirt/libvirtd.conf /etc/libvirt/libvirtd.conf.orig
sed -i 's|#listen_tls = 0|listen_tls = 0|'g /etc/libvirt/libvirtd.conf
sed -i 's|#listen_tcp = 1|listen_tcp = 1|'g /etc/libvirt/libvirtd.conf
sed -i 's|#tcp_port = "16509"|tcp_port = "16509"|'g /etc/libvirt/libvirtd.conf
sed -i 's|#auth_tcp = "sasl"|auth_tcp = "none"|'g /etc/libvirt/libvirtd.conf
cp /etc/sysconfig/libvirtd /etc/sysconfig/libvirtd.orig 
sed -i 's|#LIBVIRTD_ARGS="--listen"|LIBVIRTD_ARGS="--listen"|'g /etc/sysconfig/libvirtd
```

Restart dịch vụ
```sh 
systemctl restart libvirtd
systemctl restart openstack-nova-compute.service
```

Kiểm tra tính năng `live-migrate `
```sh 
nova live-migration $vm-id $compute_node
```

Trong đó : 
    - $vm-id : ID của máy ảo muốn migrate
    - $compute_node : hostname của node compute đích

Thực hiện login vào máy ảo và kiểm tra.


# Tài liệu tham khảo

- https://docs.openstack.org/nova/pike/admin/ssh-configuration.html