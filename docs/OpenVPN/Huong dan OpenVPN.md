# Hướng dẫn cài đặt OpenVPN

-   **Cần có:**
    - 	file  config  VPN  cho  client  (\*.ovpn)
    - 	username/password  đăng  nhập  vào  OpenVPN

-	**Lưu ý:**
    -   Server là máy muốn VPN đến, client là máy ở nhà

## Bước 1: Cài  đặt  và  chạy  OpenVPN trên máy client

### Đối với máy client là windows

1. 	Download OpenVPN cho windows:
	-	[https://openvpn.net/client-connect-vpn-for-windows/](https://openvpn.net/client-connect-vpn-for-windows/)
    -	Chọn phiên bản V3 (Beta), **không  chọn bản khác**
2. 	Cài đặt OpenVPN
3. 	Mở  “OpenVPN  Connect” chọn  tab  “IMPORT FROM  FILE”
4. 	Kéo thả file config (.ovpn)
5. 	Nhập  username và password
	-	Tick vào “Save password” và nhập password nếu không muốn nhập password cho lần sau
6. 	Chọn “Add” và connect


### Đối với máy client là Linux (CentOS)

-	Làm theo hướng dẫn ở link sau:
	-	[https://community.openvpn.net/openvpn/wiki/OpenVPN3Linux](https://community.openvpn.net/openvpn/wiki/OpenVPN3Linux)

## Bước 2: Cho phép máy client connect đến máy server (thiết lập trên máy server)

-   Có 2 giao thức phổ biến trên máy server cho phép client kết nối từ xa:
    - **VNC**:
    	- 	Giao thức tích hợp sẵn trong Linux, thường dùng cho client và server là Linux
    	- 	Cách thể hiện giống TeamViewer: thấy hoạt động trên màn hình, trên server phải login...
    	- 	**Cài đặt:**
		    1.  Từ giao diện Linux, vào `Settings  →  Sharing`
		    2.  Cho phép *Screen sharing*, *Remote Login* (SSH), sau đó đặt password

	        ![Allow connect when using VNC protocol](https://drive.google.com/uc?id=15T6a7__pYqTgB4rD239RaMsl14fI2mMa)

		    3.  Mở TCP port

				```bash
				sudo firewall-cmd --zone=public --add-port=5900/tcp --permanent
				sudo firewall-cmd --reload
				sudo firewall-cmd --list-all
				```

    - **XRDP**:
	    - 	Giao thức tích hợp sẵn trên Windows, thường dùng cho client và server là windows
	    - 	Ưu điểm: không thấy hoạt động trên màn hình, server logout thì vẫn remote vào được. Tuy nhiên muốn dùng cho server là Linux (CentOS) thì phải cài đặt giao thức XRDP
	    - 	**Cài đặt:**
		    1.  Cài đặt giao thức XRDP
			    - 	Install  the  EPEL  repository

					```bash
					sudo yum  install epel-release
					hoặc
					sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
					```

        		- 	Install XRDP => `sudo yum install xrdp tigervnc-server`
    			- 	Start service và cho phép tự chạy khi reboot

					```bash
					sudo systemctl start xrdp.service
					sudo systemctl enable xrdp.service
		            ```

			    - 	Check XRDP lắng nghe trên port 3389 => `netstat -antup | grep xrdp`
    			- 	Mở TCP port

					```bash
					sudo firewall-cmd --zone=public --add-port=3389/tcp --permanent
					sudo firewall-cmd --reload
					sudo firewall-cmd --list-all
					```

## Bước 3: Từ client kết nối đến server
### Đối với máy client là Linux (CentOS)
Từ giao diện Linux, *Applications* --> *Utilities* --> *Remote Desktop Viewer*

![linux-connect](https://drive.google.com/uc?id=1VUrf2xjfY9F7zwooS7WasI0_2qVmHJ1o)


### Đối với máy client là Windows
Mở ứng dụng *Remote Desktop Connection*:

![windows-connect](https://drive.google.com/uc?id=1EMdetg3fA7jmSRMup0Kwumk_AwjGwCjo)
