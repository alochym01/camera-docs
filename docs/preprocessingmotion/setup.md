# Queue for motion
## Component
-   Logstash
-   flask web framework
-   rq queue for executing job
    -   redis as broker for queue
        -   Edit file `/etc/redis.conf` with content
            -   maxmemory 2gb # for queue only
            -   maxmemory-policy volatile-ttl
        -   check value
            ```bash
            127.0.0.1:6379[1]> HGETALL rq:job:662a55a9-a7fc-41a9-b63f-9893340a932a
            1) "origin"
            2) "default"
            3) "status"
            4) "queued"
            5) "created_at"
            6) "2019-08-12T11:09:45.864601Z"
            7) "description"
            8) "app.task.thumbnails({'vsftpd_pid': 'pid 49239', 'cameraserial': 'camtest', 'vsftpd_file_size': '3246064 bytes', '@timestamp': '2019-08-12T11:09:44.717Z', 'offset': 7199333, 'prospector': {'type': 'log'}, 'host': {'name': 'ftp-01-hcm.fcam.vn'}, 'source': '/var/log/vsftpd.log', 'module': 'Vsftp_vid', 'log': {'file': {'path': '/var/log/vsftpd.log'}}, 'recordtimestampftpv': '2019-08-12 18.09.05', 'vsftpd_file_transfer_speed': '4228.30Kbyte/sec', 'vsftp_path': '/data/ftp/camtest/192.168.1.119_32769a4bd4a7/2019-08-12/01/rec/18.09.05-18.09.37[M][@f94][0].h264', 'beat': {'hostname': 'ftp-01-hcm.fcam.vn', 'name': 'ftp-01-hcm.fcam.vn', 'version': '6.8.2'}, 'vsftpd_action_status': 'OK', 'vsftpd_client_ip': '42.118.242.153', '@version': '1', 'daterecordv': '2019-08-12', 'vsftpd_action': 'UPLOAD', 'ftplogdate': 'Mon Aug 12 18:09:40 2019', 'recordtimestamp': '2019-08-12T11:09:05.000Z', 'input': {'type': 'log'}, 'timerecordv': '18.09.05'})"
            9) "enqueued_at"
            10) "2019-08-12T11:09:45.864915Z"
            11) "timeout"
            12) "180"
            13) "data"
            14) "x\x9cuR\xcbn\xd3@\x14-4M\xd3\x94B!,\x10+/\xb3\xc9<l\xc7\x89\xbbJ%V\x94R\x16\xc0\x82(\xb2&\xf6\xb8\xb6\xf0K\x9e\xb1\xa5V\xaa\xc4\x86\xdd,\x87\x9f\xe3k\xb8\x93\xa4\t\xa0\xd6\x0b{|\xe7>\xce9\xf7\xfc\xe8\xfc\xba\xdd\xdf[=C5`U\x85$\x13\xdf\x91L\x9a|Y\xb04\x13\xfa\xe3\x9d\x1e\xaa~+bYEA\x95FZ\x1d\xc1\xdbr}\xdb\xf1\xb5z\x16\xb2\x9c\xd7L\xf0:e\x99V\x87\xf0+\xb9\x90Z\x9dn*\xe24\xe3\x81Ho\xb9V'\x8e\xedz\xc4s\xad\xe5\r\xe4h\xd5\x9f\xc94\x87d\x96WZ\xbd\xb1\t\xf5Gd:\xa2\xf6gJ\xcf\x88\x7f\xe6\xbahB'\xdf\xb4\xea\x96q,\xb8\xd4\xef\xf9\xef|O\xf5\xab\xba\x14\x15\x0feY\xeb;\xad:\xf2\xa6\x82\xde\xfbYy\xad\x85\xea$%\x0c7\xe1\x02pi\xf5\n0\x8c\b\x1d%a\x8eb\xc0\x86\xda\x02\xb2\xba\xa2l\xea\x10\xae\a\xb8e5\x86R\xbcF\x8bL\x17\xd5\xcd\xcb\xa8\xc9\xe0\xfa\xe8\xab\x89\x06\xad!\xbd\x1a`\x1a\x1bB\xabC\xc5d\xf2H\x0b!\xd4\xa0\xe6aYG[\x86p\xd7B\xf6\x8e\xa5E\xa7\x88\xf8\x88\x8c\xb5z\xfb\xb7V\xb2f\x85\x88y\x1d\x00I\x0e\x83O]\xdb\x9e\"\x87\\\x18\xd5\xb0\xe0\xa1\xdel#X\x03`8b\x92a\b\xe0\x8d\xf8\x98\xfa6\xa2\xde\x14QD\xa9\x1f8\xf6\xc4\xf3\x99\xbb\x8c\\6\xc1\xbb\xf9\x98P\x0c\x18\xf1=\x8c\xd1\xfa\xe0L\xe6\x97\x8b\xf9,\xf6\xdd\xc5\x9c,Pb{.p]rfd\x1d\xaa\x9e\x11\xf8qm\x93\x97\x0f\x86\xd5a\xcbk\x91\x96p:\xf0\xd0\x14\xd9\xbaQ\xaf7\xa4Y(\xe1\"\x00\x91d\x03\xa6xzu\xb13O\x98\xa5\xbc\x90A\n\xfex\xee\x02'\x80h\x9b\xef\xd8\xd1\xaa7\xdb\xf6|B\xb5:\x06\x15\xf8Zs\xd0\xb9\xbf\xe3\t\xbe\xfbg\x12\xec\xf7\xcb\xa7\x0fW\xe7\xef \x0b\xc2\xb0.S\t\x06\xbc,\x0b\xeb\xbc\xb9\xb6V\xabY\x19\x90X\xa6\x8dV/\xfe\xdb\xe5Cn%cD\b\x01\xb7\x1e\xa4E\xd5\x18\xb1\x92\x93{W\x1e\x9b\xca-\xb6\xdev\xf1\xcdOH\x93\x1a\xfd\x01\x8b\xa0;\x8a"
            ```
-   elasticsearch as images database
### RQ queue
-   Any Python function can be invoked asynchronously, by simply pushing a reference to the function and its arguments onto a queue
-   This is called enqueueing
#### RQ Installation
-   pip install rq redis
-   version redis==3.3.6 rq==1.1.0
#### RQ Running and Config
-   `worker.py` is python rq to run task in a queue
-   [rq systemd](https://python-rq.org/patterns/systemd/)
-   `/etc/systemd/system/rqworker@one.service`
    ```bash
    [Unit]
    Description=RQ Worker Number %i
    After=network.target

    [Service]
    Type=simple
    WorkingDirectory=/path/to/working_directory
    Environment=LANG=en_US.UTF-8
    Environment=LC_ALL=en_US.UTF-8
    Environment=LC_LANG=en_US.UTF-8
    ExecStart=/path/to/rq worker -c worker.py
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
-   running a number of worker
    -   copy rqworker@one to rqworker@two, rqworker@three
    -   start all rqworker by systemd
        -   systemctl start rqworker@*
    -   start one rqworker by systemd
        -   systemctl start rqworker@one
### Flask framework
-   Act as a creator job queue
-   folder structure
    ```bash
    flask
    ├── app
    │   ├── __init__.py
    │   ├── task.py
    │   └── views
    │       └── queue.py
    ├── instance
    │   └── config.py
    ├── run.py
    └── worker.py
    ```
    -   `app/task.py`:
        -   is executing all logic
        -   The thumnails function will be added into a queue by flask
    -   `worker.py` is python rq to run task in a queue
### Logstash
#### Logstash plugins
-   logstash-output-http is used as a source's client for flask app
#### Logstash plugin installation
-   create `/etc/yum.repos.d/logstasg.repo`
    ```bash
    [logstash-6.x]
    name=Elasticsearch repository for 6.x packages
    baseurl=https://artifacts.elastic.co/packages/6.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md

    ```
-   yum install -y logstash
-   sudo /usr/share/logstash/bin/logstash-plugin install logstash-output-http
#### Logstash configuration
-   `/usr/share/logstash/bin/logstash --config.test_and_exit -f /etc/logstash/conf.d/motion-video.conf`
-   `chown -R logstash: /var/lib/logstash`
-   `/etc/logstash/conf.d/motion-video.conf`
    ```bash
    input {
    beats {
        host => "42.118.242.146"
        port => 5044
    }
    }
    filter {
        grok {
            match => {
                message => '(?<logdate>(?>\d\d){1,2}-(?:0?[1-9]|1[0-2])-(?:(?:0[1-9])|(?:[12][0-9])|(?:3[01])|[1-9]) (?:2[0123]|[01]?[0-9]):(?:[0-5][0-9]):(?:(?:[0-5]?[0-9]|60)(?:[:.,][0-9]+)?)) - \[%{DATA:cameraserial}]\[%{DATA:recordtimestampsrs}]\[%{DATA:savepath}]\[%{DATA:image}]\[%{DATA:duration}]\[%{DATA:location}]\[%{DATA:recordlocation}]'
            }
            add_field => { "module" => "Srs" }
            remove_field => "tags"
        }

        if("Srs" not in [module]) {
            grok {
                match => {
                    message => '(?<ftplogdate>%{DAY}%{SPACE}%{MONTH}%{SPACE}%{MONTHDAY}%{SPACE}%{TIME}%{SPACE}%{YEAR})%{SPACE}\[%{DATA:vsftpd_pid}\]%{SPACE}\[%{DATA:cameraserial}\]%{SPACE}%{WORD:vsftpd_action_status}%{SPACE}%{WORD:vsftpd_action}:%{SPACE}Client%{SPACE}\"%{IP:vsftpd_client_ip}\",%{SPACE}\"(?<vsftp_path>.*\/(?<daterecord>%{YEAR}-%{MONTHNUM}-%{MONTHDAY})\/.*\/.*\/(?<timerecord>(?:[+-]?(?:[0-9]))(?:[+-]?(?:[0-9]))(?:[+-]?(?:[0-9]))(?:[+-]?(?:[0-9]))(?:[+-]?(?:[0-9]))(?:[+-]?(?:[0-9]))).*.jpg)\",%{SPACE}%{GREEDYDATA:vsftpd_file_size},%{SPACE}%{GREEDYDATA:vsftpd_file_transfer_speed}'
                }
                add_field => {
                    "module" => "Vsftp_pic"
                    "recordtimestampftp" => "%{daterecord} %{timerecord}"
                }
                remove_field => "tags"
            }
        }

        if("Vsftp_pic" not in [module]){
            grok {
                match => {
                    message => '(?<ftplogdate>%{DAY}%{SPACE}%{MONTH}%{SPACE}%{MONTHDAY}%{SPACE}%{TIME}%{SPACE}%{YEAR})%{SPACE}\[%{DATA:vsftpd_pid}\]%{SPACE}\[%{DATA:cameraserial}\]%{SPACE}%{WORD:vsftpd_action_status}%{SPACE}%{WORD:vsftpd_action}:%{SPACE}Client%{SPACE}\"%{IP:vsftpd_client_ip}\",%{SPACE}\"(?<vsftp_path>.*\/(?<daterecordv>(?>\d\d){1,2}-(?:0?[1-9]|1[0-2])-(?:(?:0[1-9])|(?:[12][0-9])|(?:3[01])|[1-9]))\/.*\/.*\/(?<timerecordv>(?:[+-]?(?:[0-9]+)).(?:[+-]?(?:[0-9]+)).(?:[+-]?(?:[0-9]+))).*.h264)\",%{SPACE}%{GREEDYDATA:vsftpd_file_size},%{SPACE}%{GREEDYDATA:vsftpd_file_transfer_speed}'
                }
                add_field => {
                    "module" => "Vsftp_vid"
                    "recordtimestampftpv" => "%{daterecordv} %{timerecordv}"
                }
                remove_field => "tags"
            }
        }

        mutate { remove_field => "message" }
        date {
            match => ["recordtimestampsrs", "YYYY-MM-dd HH:mm:ss"]
            target => "recordtimestamp"
        }
        date {
            match => ["recordtimestampftp", "YYYY-MM-dd HHmmss"]
            target => "recordtimestamp"
        }
        date {
            match => ["recordtimestampftpv", "YYYY-MM-dd HH.mm.ss"]
            target => "recordtimestamp"
        }
    }
    output {
        if "_grokparsefailure" in [tags] {
            file { path => "/var/log/logstash/parsefailure.txt" }
        } else {
            if ("Vsftp_vid" not in [module]) {
                elasticsearch {
                    hosts => "42.118.242.146:9200"
                    manage_template => false
                    index => "camipc_%{[cameraserial]}"
                    document_type => "camerarecord"
                }
            } else {
                http {
                    url => "http://127.0.0.1:6060"
                    http_method => "get"
                    format => "json"
                    content_type => "application/json"
                }
            }
        }
    }
    ```
