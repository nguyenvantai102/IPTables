# Iptables
## Mô hình triển khai
![Untitled](https://user-images.githubusercontent.com/83420725/139803255-2183677a-7616-449f-8c9c-3445198ac978.png)

## Demo

- Squid Proxy: https://www.youtube.com/watch?v=rQqNSkvLsTY&t=1s ​

- IPtables Rules: https://www.youtube.com/watch?v=ycabV96YTGA&t=1s 

## Triển khai 

- Iptables là một ứng dụng firewall, được cài đặt mặc định trên hầu hết các phiên bản của các distro lõi Linux. Sau khi iptables được cài đặt mà chưa cấu hình, nó cũng mặc định cho phép tất cả các gói tin qua lại bình thường mà không cản trở gì cả.


- Các tùy chọn chỉ định thông số IPtables
	- Chỉ định tên table: -t ,
	- Chỉ định loại giao thức: -p ,
	- Chỉ định card mạng vào: -i ,
	- Chỉ định card mạng ra: -o ,
	- Chỉ định địa chỉ IP nguồn: -s <địa_chỉ_ip_nguồn>,
	- Chỉ định địa chỉ IP đích: -d <địa_chỉ_ip_đích>, tương tự như –s.
	- Chỉ định cổng nguồn: –sport ,
	- Chỉ định cổng đích: –dport , tương tự như –sport

- Các tùy chọn có thể thao tác với chain
	- Tạo chain mới: IPtables -N
	- Xóa hết các rule đã tạo trong chain: IPtables -X
	- Đặt chính sách cho các chain `built-in` (INPUT, OUTPUT & FORWARD): IPtables -P , ví dụ: IPtables -P INPUT 
	  ACCEPT để chấp nhận các packet vào chain INPUT
	- Liệt kê các rule có trong chain: IPtables -L
	- Xóa các rule có trong chain (flush chain): IPtables -F
	- Reset bộ đếm packet về 0: IPtables -Z



- Các tùy chọn thao tác rule trong IPtables
	- Thêm rule: -A (append)
	- Xóa rule: -D (delete)
	- Thay thế rule: -R (replace)
	- Chèn thêm rule: -I (insert)
- Phân biệt giữa ACCEPT, DROP và REJECT packet
      - ACCEPT: chấp nhận packet
      - DROP: thả packet (không hồi âm cho client)
      - REJECT: loại bỏ packet (hồi âm cho client bằng một packet khác)
## Set up 

- Stop firewalld service

      $ sudo systemctl stop firewlld
- Disable the FirewallD service to start automatically on system boot: 
      
      $ sudo systemctl disable firewalld
- Mask the FirewallD service to prevent it from being started by another services:
      
      $ sudo systemctl mask --now firewalld
- Save iptables config

      $ service iptables save
      
- List out all of the active iptables rules with numeric lines and verbose
      
      $ iptables -n -L -v --line-numbers
      
- List Rules as Tables for INPUT chain
      
      $ iptables -L INPUT -v
 
- Delete Rule by Chain and Number
      
      $ iptables -D INPUT 10
      $ iptables -D INPUT -m conntrack --ctstate INVALID -j DROP

## Config
- Cấu hình cho phép kết nối ssh
	
		iptables -A INPUT -j ACCEPT -p tcp -s 192.168.1.5 --dport 22
		iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name ssh --rsource
		iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent ! --rcheck --seconds 60 --hitcount 4 --name ssh --rsource -j ACCEPT 
- SSH brute-force protection
    - Use Hydra tool in kali linux and rockyou wordlist
        
          $ hydra -l root -P ~/Desktop/rockyou.txt 192.168.1.9 ssh
     
     ![Untitled 2](https://user-images.githubusercontent.com/83420725/139845498-5cac154d-5af8-4e12-9246-bae475cc2c76.png)

          
- Drop Invalid Packets in IPtables

           $ iptables -A INPUT -m conntrack --ctstate INVALID -j DROP 

- Thực hiện NAT 192.168.100.0/24 

      $ iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -d 0/0 -o enp0s3 -j MASQUERADE
- Cho phép bên ngoài kết nối đến webserver
	
		iptables -t nat -A PREROUTING -d 192.168.1.10  -p tcp -m tcp --dport 80 -j DNAT --to-destination 192.168.200.210
		iptables -A FORWARD -i enp0s3 -o enp0s9 -p tcp -m tcp --dport 80 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
		iptables -A FORWARD -i enp0s9 -o enp0s3 -j ACCEPT
		
		iptables -A FORWARD -i enp0s8 -o enp0s9 -p tcp -m tcp --dport 80 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
		iptables -A FORWARD -i enp0s9 -o enp0s8 -j ACCEPT

- Thiết lập rule cho phép phing ra ngoài

      $ iptables -I FORWARD -j ACCEPT -i enp0s8 -o enp0s3 -p icmp -s 192.168.100.0/24 --icmp-type echo-request -d 0/0
      $ iptables -I FORWARD -j ACCEPT -i enp0s3 -o enp0s8 -p icmp --icmp-type echo-reply
 
      $ iptables -I FORWARD -j ACCEPT -i enp0s8 -o enp0s3 -p tcp -s 192.168.100.0/24 -m state --state NEW,ESTABLISHED -d 0/0 --dport 80
      $ iptables -I FORWARD -j ACCEPT -i enp0s8 -o enp0s3 -p tcp -s 192.168.100.0/24 -m state --state NEW,ESTABLISHED -d 0/0 --dport 443
      $ iptables -I FORWARD -j ACCEPT -i enp0s8 -o enp0s3 -p udp -s 192.168.100.0/24 -m state --state NEW,ESTABLISHED -d 0/0 --dport 53

      $ iptables -I FORWARD -j ACCEPT -i enp0s3 -o enp0s8 -m state --state ESTABLISHED,RELATED


- Filter website facebook  (https: prevent 80 and 443)
      
      $ iptables -I FORWARD -i enp0s8 -o enp0s3 -p tcp --dport 80 -m string --string "facebook.com" --algo bm -j DROP
      $ iptables -I FORWARD -i enp0s8 -o enp0s3 -p tcp --dport 443 -m string --string "facebook.com" --algo bm -j DROP
      
      iptables -I FORWARD -m time --timestart $(date -u -d @$(date "+%s" -d "19:00") +%H:%M) --timestop $(date -u -d @$(date "+%s" -d "08:13") +%H:%M) --weekdays Mon,Tue,Wed,Thu,Fri  -m string --string "facebook.com" --algo bm -j DROP
      
- Cau hinh an toan cho mang ben trong
	
		INTIF="enp0s8"
		INTIP="192.168.100.100"
		INTNET="192.168.100.0/24"

		EXTIF='enp0s3'
		EXTIP='192.168.1.9'
		EXTNET=any/0

		LOIF="lo"
		LOIP="127.0.0.1"

- Cấu hình cho squid
	
		iptables -t nat -A PREROUTING -i enp0s8 -p tcp  --dport 80 -j REDIRECT --to-port 3128
		iptables -A INPUT -i enp0s8 -p tcp --dport 3128 -m state --state NEW,ESTABLISHED -j ACCEPT
		iptables -A INPUT -i enp0s3 -p tcp  --sport 80 -m state --state ESTABLISHED -j ACCEPT 
		iptables -A OUTPUT -o $INTIF -p tcp -s $INTIP --sport 8080 --dport $HI_PORTS -m state --state ESTABLISHED -j ACCEPT 
		iptables -A OUTPUT -o $EXTIF -p tcp -s $EXTIP --sport $HI_PORTS -d $EXTNET --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

	
