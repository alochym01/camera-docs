### Hướng dẫn cách cài đặt vsFTP

1.  https://www.linode.com/docs/uptime/logs/use-logrotate-to-manage-log-files/
2.  OS sử dụng CentOS 7
    *   Cài đặt
        *   repo epel **yum install -y epel-release**
        *   vsftp và các modules liên quan **yum install -y pam\_url vsftpd pam\_script**
            *   vsftp
                *   Virtual User là gì
                    *   Không phải users của OS system level
                    *   Users chỉ có ý nghĩa với FTP
            *   [pam\_url](https://github.com/mricon/pam_url)
                *   Dùng để chứng thực thông qua web services
            *   [pam\_script](https://github.com/jeroennijhof/pam_script)
                *   Dùng để tạo FTP user home folder
                *   pam\_script folder **/etc/pam-script.d/**
                *   pam\_script file execute **/etc/pam-script.d/pam\_script\_auth**
    *   Cấu hinh
        *   Disable selinux
#### Installation
-	Base repo
	-	`yum install -y epel-release`
-	Envoy
	- 	repo
		```
		[tetrate-getenvoy-stable]
		name=Tetrate GetEnvoy - Stable
		baseurl=https://tetrate.bintray.com/getenvoy-rpm/centos/$releasever/$basearch/stable/
		enabled=1
		gpgkey=https://getenvoy.io/gpg
		gpgcheck=1
		repo_gpgcheck=1
		```
	-	`yum install getenvoy-envoy`
-	FTP
	-	`yum install -y pam_url vsftpd pam_script`
	-	[pam_url](https://github.com/mricon/pam_url)
	-	[pam_script](https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/p/pam_script-1.1.8-1.el7.x86_64.rpm)
		-	http://t3chnick.blogspot.com/2011/12/vsftp-mysql-virtual-with-auto-create.html
	-	create a vsftpd account with home directory `/data/ftp`
	    ```
		mkdir /data/ftp									#create root directory for all ftp user
		mkdir /etc/vsftpd/vconf							#create vsftpd virtual users directory config
	    useradd -s /sbin/nologin -d /data/ftp vsftpd	#create a local virtual user
		chown -R vsftpd:vsftpd /etc/vsftpd/vconf
		chown -R vsftpd:vsftpd /data/ftp				#change owner root directory for all ftp user
	    ```
-	Nginx
	-	`yum install -y nginx`
	-	create a folder which store all nginx cache
		-	`mkdir /data/vsftpd`
		-	`chown -R nginx: /data/vsftpd`
-	Flask
	-	using python 3
	-	flask modules
		```
		alembic==1.0.8
		astroid==2.2.5
		atomicwrites==1.3.0
		attrs==19.1.0
		certifi==2019.3.9
		chardet==3.0.4
		Click==7.0
		coverage==4.5.3
		Flask==1.0.2
		Flask-Migrate==2.4.0
		Flask-Redis==0.3.0
		Flask-SQLAlchemy==2.3.2
		gunicorn==19.9.0
		httpie==1.0.2
		idna==2.8
		isort==4.3.15
		itsdangerous==1.1.0
		Jinja2==2.10
		lazy-object-proxy==1.3.1
		logger==1.4
		Mako==1.0.7
		MarkupSafe==1.1.1
		mccabe==0.6.1
		more-itertools==6.0.0
		pluggy==0.9.0
		prometheus-client==0.6.0
		prometheus-flask-exporter==0.6.0
		py==1.8.0
		Pygments==2.4.2
		pylint==2.3.1
		pytest==4.3.1
		pytest-cov==2.6.1
		python-dateutil==2.8.0
		python-editor==1.0.4
		redis==3.2.0
		requests==2.21.0
		six==1.12.0
		SQLAlchemy==1.3.1
		typed-ast==1.3.1
		urllib3==1.24.1
		Werkzeug==0.14.1
		wrapt==1.11.1
		```
