# Hướng dẫn sử dụng iLO web interface
## Bước 1: SSH và mở google-chrome trên server 143
```
ssh -Y tridm9@x.x.x.143
google-chrome
```
**Lưu ý**: Nếu báo lỗi google-chrome đang mở trên máy khác thì thực hiện lệnh sau: `killall chrome`

## Bước 2: Đăng nhập vào iLO
- Nhập địa chỉ IP của port iLO
	- Ví dụ: server 130 thì IP port iLO là 192.168.2.130
	- Dãy IP cho port iLO: 192.168.2.130 --> 192.168.2.156

- Nhập username và password

![enter image description here](https://drive.google.com/uc?id=1PJsvg8vp2SrMhIOj_1dRn8Ct7aVDEl2j)


## Bước 3: Sử dụng các tính năng trên iLO

### Remote Console: Tính năng vào màn hình của server

- Vào các tab *Information* --> *Overview*, ở dòng *Integrated Remote Console* chọn *HTML5*

![enter image description here](https://drive.google.com/uc?id=1zgOzh8YkdZoyRkfVppOF8PAkiiqFUupy)


#


- Màn hình remote console hiển thị, bấm *Enter* nếu vẫn chưa vào được màn hình của server

![enter image description here](https://drive.google.com/uc?id=1GL-dAjxYmVlTLoSFybEpsvM5w0Z3XBXf)

#

### Các tính năng khác
![enter image description here](https://drive.google.com/uc?id=1fDH3Uuo5CtFFJvDOSa_-mSb_AyDwIFug)


![enter image description here](https://drive.google.com/uc?id=1V7zhbPOYGSs1D9RGGf55NQFvISyil6Da)


![enter image description here](https://drive.google.com/uc?id=1Ua1H7VeQKEifl6dOzieCvLGm2W13PB_g)
