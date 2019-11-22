# Lưu ý cài đặt vận hành 

## Ceph vs OpenStack 
- IOPS thấp kiểm tra `virt-type` cấu hình nova
- Volume mới tạo ko đúng chuẩn child clone từ Images `direct_url_...` phải nằm trong `[DEFAULT]`
- Nova compute fillter không đều các node `nova-scheduler` ==> 