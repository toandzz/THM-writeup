# [THM] RootMe write-up

- IP: 10.10.76.27

## Recon

Sử dụng nmap để dò quét các cổng và dịch vụ đang mở
> nmap -sT -sV -sC -T4 10.10.76.27

![Scan nmap](./img/nmap-scan.png)

Có 3 port dịch vụ đang mở bao gồm
- Port 22 chạy dịch vụ SSH
- Port 80 và 445 chạy dịch vụ HTTP 

Ta thấy port 80 và 445 đều chạy dịch vụ http và đều có giao diện là default page apache2 nên tiếp tục scan bằng công cụ fuff để tìm thư mục ẩn 
> ffuf -u "http://10.10.76.27/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/big.txt

![ffuf scan port 80](./img/ffuf-scan80.png)
Ta nhận thấy có các thư mục ẩn `/admin`,`/passwd`,`/shadow` cùng kiểm tra các thư mục đó

Tìm thấy file id_rsa trong thư mục `/admin` có nội dung như hình ảnh bên dưới. Có thể đoán được là chuỗi này đã được mã hóa bởi thuật toán nào đó, ta sẽ thử thuật toán phổ biến nhất đó là base64 xem có gì không
![ID_RSA](./img/id_rsa.png)

![Encode id rsa](./img/id_rsa_encoded.png)
Chúng ta đã đoán đúng nó được mã hóa bằng base64 nhưng kết quả nhận được lại hơi thất vọng

Kiểm tra thư mục `/passwd`,`/shadow` có chung nội dung và khi giải mã nội dung tương tự như trên
![Encode shadow](./img/shadow-encoded.png)

Vậy thì ta sẽ tiếp tục scan với port 445
> ffuf -u "http://10.10.76.27:445/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/big.txt

![ffuf scan port 445](./img/ffuf-scan445.png)
Ta đã phát hiện ra thư mục ẩn `/management`

Giao diện thư mục ẩn `/management`
![Web interface](./img/web-interface.png)

Phát hiện ra trang `/login` ta sẽ thử attack vào các ô input với các lỗ hổng cơ bản như XSS hay SQLi
![Login interface](./img/login-interface.png)

## Exploit
Sau khi thử thì đã thành công với SQLi payload `admin'or 1=1-- -` và mật khẩu bất kì ta có thể truy cập vào dashboard của administrator
![Admin interface](./img/admin-interface.png)