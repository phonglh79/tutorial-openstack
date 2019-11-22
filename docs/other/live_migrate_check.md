# Check live Migrate khi tiến hành setup compute mới 

## Các nguyên nhân có thể dẫn đến việc Live Migrate thất bại 

- Khác loại CPU 
- Cùng loại CPU nhưng khác version BIOS main 
- Cùng CPU nhưng khác OS feature của CPU có thể khác 

## Cách kiểm tra 

Kiểm tra bằng cách `compare cpu capabilities` để kiểm tra khả năng live-migration giữa 2 compute

Trên node compute mới thực hiện cài đặt libvirt nếu chưa có
```sh
yum install libvirt -y
```

Start libvirt
```sh
systemctl start libvirtd
```

Dump capabilities ở `CÁC NODE COMPUTE còn lại `
```sh 
virsh capabilities > virsh-caps-COMPUTEx.xml
```

Chỉnh sửa file, xóa hết các dòng và chỉ giữ lại block `<cpu> </cpu>` như ví dụ dưới đây
```sh
<cpu>
      <arch>x86_64</arch>
      <model>core2duo</model>
      <topology sockets='1' cores='4' threads='1'/>
      <feature name='lahf_lm'/>
      <feature name='sse4.1'/>
      <feature name='xtpr'/>
      <feature name='cx16'/>
      <feature name='tm2'/>
      <feature name='est'/>
      <feature name='vmx'/>
      <feature name='ds_cpl'/>
      <feature name='pbe'/>
      <feature name='tm'/>
      <feature name='ht'/>
      <feature name='ss'/>
      <feature name='acpi'/>
      <feature name='ds'/>
</cpu>
```

Download file này về compute mới

Tiến hành compare
```sh 
virsh cpu-compare virsh-caps-COMPUTEx.xml
```

Kết quả ra là `superset` hoặc `identical` là tương thích

<img src="https://i.imgur.com/wzQ1XHk.png">
