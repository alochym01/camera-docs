Giới thiệu
==========

Playback service là service phục vụ khách hàng muốn xem lại các loại media như: video full record, motion record và thumbnails

Table of contents
=================

* [Giới thiệu](#giới-thiệu)
* [Flow tổng quan](#Flow-tổng-quan)
* [Playback API](#Playback-API)
    * [Source code tree playback api](#Source-code-tree-playback-api)
    * [Các thành phần tham gia playback api](#Các-thành-phần-tham-gia-playback-api)
    * [Cài đặt và các command playback api](#Cài-đặt-và-các-command-playback-api)
        * [Playback flask](#playback-flask)
        * [ElasticSearch](#ElasticSearch)
        * [Redis (playback api)](#Redis-playback-api)
    * [Audit playback api](#Audit-playback-api)    
* [Playback service](#Playback-service)
    * [Source code tree playback service](#Source-code-tree-playback-service)
    * [Các thành phần tham gia playback service](#Các-thành-phần-tham-gia-playback-service)
    * [Cài đặt và các command playback service](#Cài-đặt-và-các-command-playback-service)
        * [Envoy proxy](#Envoy-proxy)
        * [OpenResty](#OpenResty)
        * [Redis (playback service)](#Redis-playback-service)
        * [Nginx](#Nginx)
    * [Audit playback service](#Audit-playback-service)
        * [File config envoy](#File-config-envoy-/etc/envoy-ftp.yaml)
        * [File config Nginx của OpenResty](#File-config-Nginx-của-OpenResty-/usr/local/openresty/nginx/conf/nginx.conf)
        * [File initialize lua source](#File-initialize-lua-source-/usr/local/openresty/nginx/luasource/initialize.lua)
        * [File checkmotion/checkplay lua source](#File-checkmotion/checkplay-lua-source-/usr/local/openresty/nginx/luasource/checkmotion.lua-or-.../checkplay.lua)
        * [File config Redis](#File-config-Redis-/etc/redis.conf)

Flow tổng quan
==============
![playback-flow](./images/playback-flow.jpg)

(sửa lại hình và giải thích flow sau)

[Back to TOC](#table-of-contents)

Playback API
============

Nhiệm vụ: tiếp nhận thông tin từ API Customer, xử lí, tìm kiếm và trả về danh sách các đường link để playback

![playbackapi-flow](./images/playbackapi-flow.jpg)

### Input:

* Cam serial: muốn xem playback cam nào
* Start time: thời gian bắt đầu
* Stop time: thời gian kết thúc
    * Hiện tại start time là 0h, stop time là thời điểm yêu cầu xem (của ngày cần xem)
* Type of request: muốn xem full record hay motion

    Ví dụ:
   
        {
        "camserial": "ggfd43vfsdfdsfa",
        "clientip": "x.x.x.x",
        "m_starttime": "2019-10-23T00:00:00",
        "m_stoptime": "2019-10-23T00:50:20"
        }
    

### Output: 

* Danh sách các đường link 

    Ví dụ

    * Nếu request type là motion
        
            {
            "motionpic": [
                {
                "file": "http://x.x.x.x/iwantmotion/?token=573e2115d72219b6d3b6f46beff22f41f0a239bdd092592b96c88753033f4106&pos=35",
                "recordtimestamp": "2019-12-16T22:50:25.000Z"
                },
                {
                "file": "http://x.x.x.x/iwantmotion/?token=573e2115d72219b6d3b6f46beff22f41f0a239bdd092592b96c88753033f4106&pos=36",
                "recordtimestamp": "2019-12-16T23:20:45.000Z"
                }
            ],
            "motionvid": [
                {
                "file": "http://x.x.x.x/iwantmotion/?token=573e2115d72219b6d3b6f46beff22f41f0a239bdd092592b96c88753033f4106&pos=1",
                "recordtimestamp": "2019-12-16T22:50:25.000Z",
                "thumbnail": "http://x.x.x.x/iwantmotion/?token=573e2115d72219b6d3b6f46beff22f41f0a239bdd092592b96c88753033f4106&pos=2"
                },
                {
                "file": "http://x.x.x.x/iwantmotion/?token=573e2115d72219b6d3b6f46beff22f41f0a239bdd092592b96c88753033f4106&pos=3",
                "recordtimestamp": "2019-12-16T23:20:45.000Z",
                "thumbnail": "http://x.x.x.x/iwantmotion/?token=573e2115d72219b6d3b6f46beff22f41f0a239bdd092592b96c88753033f4106&pos=4"
                }
            ],
            "result": true
            }
        

    * Nếu request type là full record
        
            {
            "recording": [
                {
                "duration": "908.276",
                "file": "http://x.x.x.x/iwanttoplay/?token=997a00135c3b15511c4551e408ca391c6adfc280921f38c6cad8e7768af1eadc&pos=1",
                "image": "https://x.x.x.x/thumbnails/01954e3dbea8a12f/1576515034491.jpg",
                "recordtimestamp": "2019-12-16T17:05:54.000Z"
                },
                {
                "duration": "908.2",
                "file": "http://x.x.x.x/iwanttoplay/?token=997a00135c3b15511c4551e408ca391c6adfc280921f38c6cad8e7768af1eadc&pos=2",
                "image": "https://x.x.x.x/thumbnails/01954e3dbea8a12f/1576515954623.jpg",
                "recordtimestamp": "2019-12-16T17:21:14.000Z"
                }
            ],
            "result": true
            }
        

[Back to TOC](#table-of-contents)

Source code tree playback api
-----------------------------

* Đuợc viết bằng python, kết hợp với flask và gunicorn
* Cấu trúc thư mục source code 

        flask-api-playback/
        ├── app
        │   ├── __init__.py 
        │   ├── models
        │   │   ├── examplemodel.py
        │   │   └── __init__.py
        │   ├── personalulti.py
        │   ├── __pycache__
        │   │   └── __init__.cpython-37.pyc
        │   └── views
        │       ├── __init__.py
        │       ├── playbackapi.py
        │       ├── __pycache__
        │       │   └── playbackapi.cpython-37.pyc
        │       └── restplusexampleapi.py
        ├── app.db
        ├── doc
        │   ├── config.md
        │   ├── Playbackapiandplayback.jpg
        │   └── troubleshoot.md
        ├── example.sql
        ├── flask-api-playback.service
        ├── gunicorn
        │   └── config.py
        ├── __init__.py
        ├── instance
        │   ├── app.db
        │   ├── config.py
        │   └── __init__.py
        ├── migrations
        │   ├── alembic.ini
        │   ├── env.py
        │   ├── README
        │   └── script.py.mako
        ├── __pycache__
        │   ├── config.cpython-37.pyc
        │   └── __init__.cpython-37.pyc
        ├── README.md
        ├── requirement.txt
        ├── run.py
        ├── systemd
        │   └── env
        └── tests
            ├── __init__.py
            ├── __pycache__
            │   ├── __init__.cpython-37.pyc
            │   └── test_api.cpython-37-PYTEST.pyc
            └── test_api.py


(giải thích nhiệm vụ folder/file + delete những file ko cần thiết sau)

[Back to TOC](#table-of-contents)

Các thành phần tham gia playback api
------------------------------------

(hình vẽ các thành phần với IP và port)

### 1. Playback API

Nhận thông tin cần xem playback, tạo mẫu query ElasticSearch, lưu dữ liệu search được vào Redis và tạo danh sách các đường link 

### 2. ElasticSearch

Công cụ giúp tìm kiếm nhanh chóng tên file cần playback theo dữ liệu đầu vào (camserial, start time, stop time) 

(link tài liệu ElasticSearch)

### 3. Redis (playback api)

Là database đệm thứ nhất với:
* Key là token, là kết quả hash từ timestamp. 1 token đặc trưng cho 1 danh sách gồm các file record được, tại 1 thời điểm 
* Value là 1 table chứa kết quả search được từ ElasticSearch, là một phần đường path để play các file. Trong table này thì vị trí đầu tiên là địa chỉ IP gửi từ API Customer 

* Ví dụ:
    
        127.0.0.1:6379> get dbab7853f028c42936c42846b2ab3f7f5073b2bb2ce126b34762d96d22e285a1

        "['x.x.x.x', '988aa1c21aec2a4e/1581432216012.mp4', '988aa1c21aec2a4e/1581433136131.mp4', '988aa1c21aec2a4e/1581434056133.mp4', '988aa1c21aec2a4e/1581434976059.mp4']"
    

[Back to TOC](#table-of-contents)

Cài đặt và các command playback api
-----------------------------------

### Playback flask

#### Cài đặt Flask và các package

Flask là một micro framework giúp tạo web server dễ dàng khi code python, thuờng dùng khi chạy dev, còn chạy production thì kết hợp thêm gunicorn

* Tạo python virtual environment 
* Cài đặt flask và các package hỗ trợ cho playback API từ file `requirement.txt`
    
        pip install -r requirement.txt
    

Tham khảo thêm: https://flask.palletsprojects.com/en/1.1.x/installation/

(Chức năng các package liên quan sẽ giải thích sau)

#### Cài đặt Gunicorn
Gunicorn giúp tạo web server chạy cho môi trường production, hỗ trợ cấu hình các thread khi chạy giúp tối ưu web server


    pip install gunicorn


#### File location

API folder: /usr/src/flask-api-playback/

#### Các command

* Chạy Flask appplication
    
        flask run 
        # mặc định sẽ tìm file “app.py” để chạy
        # để thay đổi: export FLASK_APP=<NewFile>
    
    hoặc
    
        python <file>.py
    

* Chạy Gunicorn application
    
        gunicorn [options] APP_MODULE
        # APP_MODULE = $(module_name):$(variable_name)
        # Example 1: gunicorn -b localhost:8880 -w 4 wsgi:app
        # Example 2: gunicorn app:app
    

[Back to TOC](#table-of-contents)

### ElasticSearch
(link tài liệu ElasticSearch)

[Back to TOC](#table-of-contents)

### Redis (playback api)

#### Install Redis

    sudo yum install epel-release # Install epel repository
    sudo yum install redis # Install Redis

    # Verify installation
    redis-cli
    127.0.0.1:6379 > ping
    # return PONG if install success


#### File location
* config file: /etc/redis.conf
* log file: /var/log/redis/redis.log
* database (if exist): /var/lib/redis

#### Các command
* systemd command
    
        systemctl enable redis
        systemctl start/stop/restart/reload redis
    

* basic command

        redis-cli
        127.0.0.1:6379 > info # Redis server info
        127.0.0.1:6379 > set <key> <value> # Redis set 
        127.0.0.1:6379 > get <key> # Redis get
        127.0.0.1:6379 > keys * # Redis get all keys
        127.0.0.1:6379 > del <key> # Redis delete
        127.0.0.1:6379 > flushall # Redis delete all
    
[Back to TOC](#table-of-contents)

#### Một số lưu ý
* Đọc thêm về [Redis persistence](https://redis.io/topics/persistence)

Audit playback api
------------------

* Remove folder/file/module for testing (db migration...)
* Tiếp tục audit code và giải thích sau

[Back to TOC](#table-of-contents)

Playback service
================

Nhiệm vụ: nhận yêu cầu playback 1 đường link nào đó từ user, xử lí và trả về path để người dùng xem playback

![playbackservice-flow](./images/playbackservice-flow.jpg)

(sẽ sửa lại hình sau: thay IP local, viết lại tên các flow)

### Input: 
* 1 đường link bất kì từ danh sách các đường link, trên đường link có các thông số: 
    * playback type: /iwantmotion/ hoặc /iwanttoplay/
    * token: chuỗi hash từ timestamp, token là key để get value trong redis table, mỗi token key get về 1 danh sách các media (motion hay record) trong 1 ngày từ 0h đến lúc request (starttime = 0h, stoptime = now)
    * pos: vị trí của media trong redis table, bắt đầu từ pos=1 (pos=0 là IP), pos càng cao -> timestamp càng lớn

    Ví dụ:

   
        /iwanttoplay/?token=774e26b84b3f23f88eb963f60b877da465ab9b798458400f83979820d24d6a57&pos=9
    

### Output: 
* Path để playback
    * Tùy theo playback type (motion hay full record) mà proxy qua port 3000 hay 3001
    * Thành phần cuối trong path sẽ tên file cần stream lưu ở server, cũng là value tương ứng với pos trong redis table

    Ví dụ:

    
        x.x.x.x:3000/content/nghia_2018-10-05-17.30.39.443-ICT_1_2018-10-05-17.45.39.420-ICT.mp4
    

[Back to TOC](#table-of-contents)

Source code tree playback service
---------------------------------

* OpenResty
    
        /usr/local/openresty/nginx/conf/
        ├── fastcgi.conf
        ├── fastcgi.conf.default
        ├── fastcgi_params
        ├── fastcgi_params.default
        ├── koi-utf
        ├── koi-win
        ├── mime.types
        ├── mime.types.default
        ├── nginx.conf # modified only
        ├── nginx.conf.default
        ├── scgi_params
        ├── scgi_params.default
        ├── uwsgi_params
        ├── uwsgi_params.default
        └── win-utf
    
        /usr/local/openresty/nginx/luasource/
        ├── checkmotion.lua
        ├── checkplay.lua
        └── initialize.lua
   

* Nginx


        /etc/nginx/
        ├── conf.d
        │   ├── nginx-motion.conf
        │   ├── nginx-mp4.conf
        │   ├── nginx-thumbnails.conf
        │   └── vsftp-nginx.conf
        ├── default.d
        ├── fastcgi.conf
        ├── fastcgi.conf.default
        ├── fastcgi_params
        ├── fastcgi_params.default
        ├── koi-utf
        ├── koi-win
        ├── mime.types
        ├── mime.types.default
        ├── nginx.conf # modified only
        ├── nginx.conf.default
        ├── scgi_params
        ├── scgi_params.default
        ├── uwsgi_params
        ├── uwsgi_params.default
        └── win-utf


[Back to TOC](#table-of-contents)

Các thành phần tham gia playback service
----------------------------------------

(hình vẽ các thành phần với IP và port)

### 1. Envoy proxy

Nhận request từ user sau đó proxy qua OpenResty

### 2. OpenResty

Xử lí đường link từ user
* Tách token và pos để tìm meta data trong Redis
    * Nếu tìm không thấy sẽ tìm trong Redis ở Playback API

* So sánh IP user gửi 1 đường link và IP user yêu cầu playback lúc đầu xem có trùng hay không

* Tạo path rồi proxy qua Nginx để playback

### 3. Redis (playback service)

Là database đệm thứ hai với:
* Lấy dữ liệu từ Redis Playback API
    * Key là token, là kết quả hash từ timestamp. 1 token đặc trưng cho 1 danh sách gồm các file record được, tại 1 thời điểm 
    * Value là 1 table chứa kết quả search được từ ElasticSearch, là một phần đường path để play các file. Trong table này thì vị trí đầu tiên là địa chỉ IP gửi từ API Customer 

    * Ví dụ:

            127.0.0.1:6379> get dbab7853f028c42936c42846b2ab3f7f5073b2bb2ce126b34762d96d22e285a1

            "['x.x.x.x', '988aa1c21aec2a4e/1581432216012.mp4', '988aa1c21aec2a4e/1581433136131.mp4', '988aa1c21aec2a4e/1581434056133.mp4', '988aa1c21aec2a4e/1581434976059.mp4']"
        

### Nginx

Tiếp nhận request từ OpenResty và cho phép playback

[Back to TOC](#table-of-contents)

Cài đặt và các command playback service
---------------------------------------

### Envoy proxy

#### Install Envoy proxy
* Install yum-utils manager utility
   
        sudo yum install yum-utils
   

* Add Envoy repository
    
        sudo yum-config-manager --add-repo https://getenvoy.io/linux/centos/tetrate-getenvoy.repo
    

* Install Envoy

        sudo yum install getenvoy-envoy
        # Install  version mới nhất
        # Start envoy sẽ bị lỗi nếu apply file config hiện tại
    

    hoặc install theo version


        yum install getenvoy-envoy-1.11.1 
        # Version đang dùng trên server


    Tham khảo các version: 
    https://tetrate.bintray.com/getenvoy-rpm/centos/7/x86_64/stable/Packages/

* Verify Envoy
    
        envoy --version
   

#### File location

* config file: /etc/envoy.yaml

#### Các command

* Start Envoy
    
        envoy -c <config_file.yaml> --mode <>

        # mode:
            # serve (default): validate config file and start envoy
            # validate: only validate config file
    

* Envoy statistics
    
        curl <ip:admin_port>/clusters
        curl <ip:admin_port>/server_info
        curl <ip:admin_port>/stats

        # admin_port: “port_value” in "admin" section in config file
    

[Back to TOC](#table-of-contents)

### OpenResty

#### Install Openresty
* Before install openresty: if nginx is already installed and running, try disabling and stopping it before installing openresty like below:
    
        sudo systemctl disable nginx
        sudo systemctl stop nginx
    

* Install yum-utils manager utility

        sudo yum install yum-utils
    

* Add OpenResty repository

        sudo yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
    

* Install OpenResty
    
        sudo yum install openresty
    

* Verify OpenResty
    
        yum list installed | grep openresty
    
    or
    
        openresty -v
        > nginx version: openresty/1.15.8.2
        # openresty version: 1.15.8.2, nginx version: 1.15.8
    

* Install các tool và library liên quan
    * Install opm tool (để install các library)
        
            sudo yum install openresty-opm

            # Các command:
                # opm list: list ra các library đã cài bằng opm
                # opm search <name>: search library name
        
    
    * Install library (sẽ giải thích chức năng các library sau)
       
            sudo opm install pintsized/lua-resty-http
            sudo opm install tomas/lua-resty-elasticsearch                                
            sudo opm install openresty/lua-resty-redis                                   
            sudo opm install openresty/lua-resty-string                                   
            sudo opm install openresty/lua-resty-lrucache                                 
            sudo opm install hamishforbes/lua-resty-iputils   
        
    
    * Install resty utility (optional)
        
            sudo yum install openresty-resty

            # command: resty
        

    * Install restydoc documentation utility (optional)
        
            sudo yum install openresty-doc

            # command: restydoc <library_name>
        

#### File location

* config file: /usr/local/openresty/nginx/conf/nginx.conf
* lua source: /usr/local/openresty/nginx/ 
* log file: /usr/local/openresty/nginx/logs/

#### Các command
* Validate config file
    
        openresty -t
    

* Operation command 
    
        systemctl enable openresty
        systemctl start/stop/restart/reload openresty

        # must restart if any change in “listen” directive
    

#### Một số lưu ý
* `worker_processes auto`: sinh ra 2n+1 processes gồm 1 master process và 2n worker process (n: cpu threads)
        
[Back to TOC](#table-of-contents)

### Redis (playback service)

#### Install Redis

    sudo yum install epel-release # Install epel repository
    sudo yum install redis # Install Redis

    # Verify installation
    redis-cli
    127.0.0.1:6379 > ping
    # return PONG if install success


#### File location
* config file: /etc/redis.conf
* log file: /var/log/redis/redis.log
* database (if exist): /var/lib/redis

#### Các command
* systemd command
    
        systemctl enable redis
        systemctl start/stop/restart/reload redis
    

* basic command
    
        redis-cli
        127.0.0.1:6379 > info # Redis server info
        127.0.0.1:6379 > set <key> <value> # Redis set 
        127.0.0.1:6379 > get <key> # Redis get
        127.0.0.1:6379 > keys * # Redis get all keys
        127.0.0.1:6379 > del <key> # Redis delete
        127.0.0.1:6379 > flushall # Redis delete all
    
#### Một số lưu ý
* Đọc thêm về [Redis persistence](https://redis.io/topics/persistence)

[Back to TOC](#table-of-contents)

### Nginx

#### Install Nginx

    sudo yum install epel-release # Install epel repository
    sudo yum install nginx # Install Nginx

    # Verify installation
    sudo nginx -V


#### File location
* config file: /etc/nginx/nginx.conf (default)
    * On server: /etc/nginx/conf.d/
        * nginx-motion.conf
        * nginx-mp4.conf
        * nginx-thumbnails.conf
        * vsftp-nginx.conf
* log file: /var/log/nginx

#### Các command
* systemd command
    
        systemctl enable nginx
        systemctl start/stop/restart/reload nginx
    

* basic command
    * Test config file
        
            nginx -t
        

#### Nginx debug
* Using cURL tool
    
        curl -I http://x.x.x.x # response (header only)
        curl -i http://x.x.x.x # response (header + body)
        curl -v http://x.x.x.x # all request and response
    

* Using `ngrep` tool
    
        ngrep -W byline port <port_number> -d any
    

* Using Postman tool
    * GUI tool to see requests and responses
    * Easily for choose HTTP method, add header and edit the body section
    * Reference: https://www.postman.com/

* Print content in Nginx (sẽ cập nhật chi tiết sau)
    * Output to logs
    * Return through header 
        
            add_header <header_name> <variable> always
            # always return, no matter what the response code
        
    * Return through body
        * HTML file
        * via `ngx.say()` function in lua-nginx-module

#### Một số lưu ý
* Nginx không thực hiện từ trên xuống dưới trong file config mà theo thứ tự các phases:
    1. rewrite: `set`
    2. access: `access_by_lua_file`
    3. content: `add_header`, `proxy_pass`

[Back to TOC](#table-of-contents)

Audit playback service
----------------------

* Xóa Nginx và chuyển về OpenResty 
* Check IP playback
* Tiếp tục audit và giải thích code sau

### File config envoy (`/etc/envoy-ftp.yaml`)


    .....................

    - address:
        socket_address:
            address: x.x.x.x
            port_value: 80
        filter_chains:
        - filters:
        - name: envoy.http_connection_manager
            typed_config:
            "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
            generate_request_id: true
            use_remote_address: true #add to get IP when require a playback link
            tracing:
                operation_name: egress
            codec_type: auto
            stat_prefix: account_server
            route_config:
                name: local_route
                virtual_hosts:
                - name: account_server
                domains: ["x.x.x.x","x.x.x.x"]
                routes:
                - match:
                    prefix: "/iwanttoplay/" #add "/" at last

    ..........................


[Back to TOC](#table-of-contents)

### File config Nginx của OpenResty (`/usr/local/openresty/nginx/conf/nginx.conf`)


    .......................

    location /iwanttoplay/ {
        set $target '';
        #lua_code_cache off; -> only use for development, remove for best performance 
        access_by_lua_file ./luasource/checkplay.lua;
        proxy_pass $target;

        proxy_redirect off;
        # Remove the following lines because unnecessary
        #proxy_set_header Host $host; 
        #proxy_set_header X-Real-IP $remote_addr;
        #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /iwantmotion/ {
        set $target '';
        #lua_code_cache off; -> only use for development, remove for best performance
        access_by_lua_file ./luasource/checkmotion.lua;
        proxy_pass $target;

        proxy_redirect off;
        # Remove the following lines because unnecessary
        #proxy_set_header Host $host;
        #proxy_set_header X-Real-IP $remote_addr;
        #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    .............................


[Back to TOC](#table-of-contents)

### File initialize lua source (`/usr/local/openresty/nginx/luasource/initialize.lua`)


    .............

    -- Remove this function because unnecessary
    function getval(param1, param2 )
        for key, val in pairs(param1) do
        if param2 == key then return param1[key]
        end
    end
    end

    ..............


[Back to TOC](#table-of-contents)

### File checkmotion/checkplay lua source (`/usr/local/openresty/nginx/luasource/checkmotion.lua or .../checkplay.lua`)


    .............

    ngx.req.read_body() -- Remove because GET method has no body

    -- Replace 2 lines
    local m_token = getval(args,"token")
    local m_pos = getval(args,"pos")

    -- By the following lines for simple
    local m_token = args["token"]
    local m_pos = args["pos"]

    ----------------------
    local res, err = red:get(m_token)
    -- Add following codes for case: fail to get token
    if not res then
    ngx.say("failed to get token value: ", err)
    return
    end

    ..............


[Back to TOC](#table-of-contents)

### File config Redis (`/etc/redis.conf`)

    ............

    # Remove the following lines because duplication 
    bind 127.0.0.1
    appendfsync everysec

    .................



[Back to TOC](#table-of-contents)
