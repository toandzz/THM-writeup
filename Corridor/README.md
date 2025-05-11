# [THM] Corridor write-up

- IP: 10.10.179.149

## Recon

Sử dụng nmap để dò quét các cổng và dịch vụ đang mở
> nmap -sT -sV -sC -T4 10.10.179.149

<p align="center">
  <img src="./img/nmap-scan.png" alt="Nmap Scan">
</p>

Có 1 port dịch vụ đang mở 
- Port 80 chạy dịch vụ http

Sau khi scan nmap có mỗi cổng dịch vụ 80 mở và kết hợp với dữ kiện đầu bài cho đó là khám phá lỗ hổng `IDOR` tiềm ẩn trong trang web. Với đó là các giá trị thập lục phân (có thể đoán được là hàm băm)

**Giao diện trang web**
<p align="center">
  <img src="./img/web-interface.png" alt="Web interface">
</p>

Sau khi kiểm tra source code thì đúng như dữ kiện đầu bài cho ta đã có được một số chuỗi có vẻ như hàm băm
<p align="center">
  <img src="./img/web-source.png" alt="Web Souorce">
</p>

Đó là link khi ta bấm vào cánh cửa ta sẽ được redirect tới đường dẫn như hình bên dưới
<p align="center">
  <img src="./img/empty-room.png" alt="Empty room">
</p>

Ta sẽ thử phân tích mã này trên [Cyberchef](https://gchq.github.io/CyberChef/)
<p align="center">
  <img src="./img/hash-detect.png" alt="Hash detect">
</p>

Với kết quả trên ta sẽ thử giải mã `MD5` với công cụ [Crackstation].(https://crackstation.net/)
<p align="center">
  <img src="./img/crackstation-7.png" alt="Crackstation 7">
</p>

Ta sẽ check bất kì cánh cửa nào khác xem sao, với kết quả như dưới ta có thể thấy được đây là các mã hash từ 1 tới 13
<p align="center">
  <img src="./img/crackstation-6.png" alt="Crackstation 6">
</p>

## Exploit
Với lỗ hổng `IDOR` thì ta có thể truy cập được các đường dẫn mà người dùng hiện tại không được phép truy cập. Với những điều mà ta đã tìm ra thì có thể chèn payload là các số thứ tự đã được băm với `MD5`

- Sử dụng công cụ `burp intruder` để gửi payload từ 1-20 mà ta đã băm
<p align="center">
  <img src="./img/burp-position.png" alt="Burp position">
</p>
<p align="center">
  <img src="./img/burp-payload.png" alt="Burp payload">
</p>

## Flag
Với payload `cfcd208495d565ef66e7dff9f98764da` là băm md5 của 0 ta đã nhận được flag
<p align="center">
  <img src="./img/flag.png" alt="Flag">
</p>


**DONE**
