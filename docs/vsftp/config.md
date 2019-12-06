#### Configuration
-	Envoy
	-	`/etc/envoy/envoy-fpt.yaml`
		```

		static_resources:
		  listeners:
		  - address:
		      socket_address:
		        address: {ip}
		        port_value: 21
		    filter_chains:
		    - filters:
		      - name: envoy.tcp_proxy
		        config:
		          stat_prefix: ftp_server
		          cluster: ftp_server
		  - address:
		      socket_address:
		        address: {ip}
		        port_value: 80
		    filter_chains:
		    - filters:
		      - name: envoy.http_connection_manager
		        config:
		          codec_type: auto
		          stat_prefix: account_server
		          route_config:
		            name: local_route
		            virtual_hosts:
		            - name: account_server
		              domains: ["ftp-hcm.fcam.vn","ftp-01-hcm.fcam.vn"]
		              routes:
		              - match:
		                  prefix: "/"
		                route:
		                  cluster: account_server
		          http_filters:
		          - name: envoy.router
		            typed_config: {}
		  clusters:
		  - name: ftp_server
		    connect_timeout: 0.25s
		    type: STATIC
		    lb_policy: round_robin
		    load_assignment:
		      cluster_name: ftp_service
		      endpoints:
		      - lb_endpoints:
		        - endpoint:
		            address:
		              socket_address:
		                address: {ip}
		                port_value: 21021
		  - name: account_server
		    connect_timeout: 0.25s
		    type: STATIC
		    lb_policy: round_robin
		    load_assignment:
		      cluster_name: account_server
		      endpoints:
		      - lb_endpoints:
		        - endpoint:
		            address:
		              socket_address:
		                address: 127.0.0.1
		                port_value: 5001
		  - name: zipkin
		    connect_timeout: 1s
		    type: strict_dns
		    lb_policy: round_robin
		    load_assignment:
		      cluster_name: zipkin
		      endpoints:
		      - lb_endpoints:
		        - endpoint:
		            address:
		              socket_address:
		                address: {ip}
		                port_value: 9411
		tracing:
		  http:
		    name: envoy.zipkin
		    typed_config:
		      "@type": type.googleapis.com/envoy.config.trace.v2.ZipkinConfig
		      collector_cluster: zipkin
		      collector_endpoint: "/api/v1/spans"
		      shared_span_context: false
		admin:
		  access_log_path: "/dev/null"
		  address:
		    socket_address:
		      address: 0.0.0.0
		      port_value: 8000

		```
	-	envoy systemd service file
		```
		[Unit]
		Description=envoy-ftp
		After=network-online.target
		Wants=network-online.target

		[Service]
		User=root
		ExecStart=/usr/bin/envoy -c /etc/envoy/envoy-ftp.yaml

		[Install]
		WantedBy=multi-user.target

		```
-	FTP
	-	`/etc/vsftpd/vsftpd.conf`
		```
		anonymous_enable=NO
		local_enable=YES
		write_enable=YES
		local_umask=022
		dirmessage_enable=YES
		# Activate logging of uploads/downloads.
		xferlog_enable=YES
		# verbose ftp log, should disable xferlog_std_format=NO
		xferlog_std_format=NO
		log_ftp_protocol=YES
		# Make sure PORT transfer connections originate from port 20 (ftp-data).
		connect_from_port_20=YES
		# You may change the default value for timing out an idle session.
		idle_session_timeout=600
		# You may change the default value for timing out a data connection.
		data_connection_timeout=120
		# When "listen" directive is enabled, vsftpd runs in standalone mode and
		# listens on IPv4 sockets. This directive cannot be used in conjunction
		# with the listen_ipv6 directive.
		listen=YES
		# change vsftpd listen port
		listen_port=21021
		listen_ipv6=NO
		# enable ls/dir commandline
		cmds_denied=LIST
		# disable PASV security check
		pasv_promiscuous=YES
		# filebeat will read the log
		vsftpd_log_file=/var/log/vsftpd.log
		nopriv_user=vsftpd
		# using pam service name
		pam_service_name=vsftpd
		userlist_enable=YES
		tcp_wrappers=YES
		allow_writeable_chroot=YES
		guest_enable=YES
		guest_username=vsftpd
		local_root=/data/ftp/$USER
		user_sub_token=$USER
		virtual_use_local_privs=YES
		user_config_dir=/etc/vsftpd/vconf
		```
	-	pam_url
		-	`/etc/pam_url.conf`
          	```
				pam_url:
				{
				    settings:
				    {
				        url         = "http://127.0.0.1:7001/account/check"; # URI to fetch
				        returncode  = "OK";                        # The remote script/cgi should return a 200 http code and this string as its only results
				        userfield   = "username";                  # userfield name to send
				        passwdfield = "password";                  # passwdfield name to send
				        extradata   = "&do=login";                 # extra data to send
				        prompt      = "Token: ";                   # password prompt
				    };

				    ssl:
				    {
				        verify_peer = true;                               # Verify peer?
				        verify_host = true;                               # Make sure peer CN matches?
				        client_cert = "/etc/pki/tls/certs/totpcgi.crt";   # Client-side certificate
				        client_key  = "/etc/pki/tls/private/totpcgi.pem"; # Client-side key
				        ca_cert     = "/etc/pki/tls/certs/ca-bundle.crt"; # ca cert - defaults to ca-bundle.crt
				    };
				};
          	```
    using pam for vsftpd /etc/pam.d/vsftpd
    ```
        # Auth in Web API
        auth       required  pam_url.so config=/etc/pam_url.conf

        # Running script if user have no root's directory 
        # the script file name shoule be pam_script_auth
        auth       required pam_script.so onerr=success dir=/etc
	
        # Account in Web API
        account    required   pam_url.so config=/etc/pam_url.conf
    ```

	-	pam_script
		-	pam_script_auth should be an executable file
			-	chmod +x `/etc/pam-script.d/pam_script_auth`
		-	`/etc/pam-script.d/pam_script_auth` with content
			```
			#!/bin/sh

			if [ ! -d "/opt/ftp/$PAM_USER" ]; then
			/usr/bin/env mkdir /data/ftp/$PAM_USER
			/usr/bin/env chown vsftpd:vsftpd /data/ftp/$PAM_USER
			/usr/bin/env chmod 755 /data/ftp/$PAM_USER
			fi
			```
-	Nginx
	-	`/etc/nginx/conf.d/ftp.conf`
		```
		proxy_cache_path /data/vsftpd levels=1:2 keys_zone=ftp:10m inactive=24h max_size=100m;

		server {
			# Public-facing cache server.
			listen 127.0.0.1:5000;
			location / {
				proxy_cache_key "$proxy_host$request_body";
				proxy_cache ftp;
				proxy_cache_methods POST;
				proxy_pass http://127.0.0.1:5001;
				proxy_cache_valid 200 24h;
			}
		}
		```
-	Flask
	-	systemd enviroments file
		```
		prometheus_multiproc_dir=/var/log/flask-ftp-account/gunicorn
		PIDFile=/var/run/flask-ftp-account/vsftpd_flask.pid
		gunicorn_config_file=/usr/src/flask-ftp-account/gunicorn/config.py
		PYTHONPATH=/usr/src/flask-ftp-account/.env/bin/python
		FLASK_CONFIG="production"
		WORKER=4
		FLASK_APP="/usr/src/flask-ftp-account/run.py"
		```
	-	systemd file
		```
		[Unit]
		Description = VsFTP for FLASK
		After = network.target

		[Service]
		Type=simple
		LimitNOFILE=65536
		EnvironmentFile=~/usr/src/flask-ftp-account/systemd/env
		WorkingDirectory=/usr/src/flask-ftp-account

		User = ansible
		Group = ansible

		# start gunicorn at flask source code
		# check cpu cores - cat /proc/cpuinfo | grep 'core id' | wc -l
		ExecStart = /usr/src/flask-ftp-account/.env/bin/gunicorn -c ${gunicorn_config_file} run:app -b 127.0.0.1:5001 -w ${WORKER} --pid ${PIDFile}
		ExecReload = /bin/kill -s HUP $MAINPID
		ExecStop = /bin/kill -s TERM $MAINPID
		ExecStopPost = /bin/rm -rf $PIDFile
		PrivateTmp = true

		[Install]
		WantedBy=multi-user.target
		```
	-	Database schema
		```
		CREATE TABLE  accounts (
			id int(11) AUTO_INCREMENT PRIMARY KEY,
			username varchar(100) NOT NULL,
			password varchar(100) NOT NULL,
			active int(11) NOT NULL,
			timestamp DATETIME NOT NULL
		);
		```
#### Flask API example
	-	URL của API
		-	URL tạo account **/account/create**
			-	Methods **POST**
			-	Parameters
				-	username
				-	password
			-	Request
				```
				POST /account/create HTTP/1.1
				Accept: */*
				Accept-Encoding: gzip, deflate
				Connection: keep-alive
				Content-Length: 31
				Content-Type: application/x-www-form-urlencoded; charset=utf-8
				Host: 127.0.0.1:5000
				User-Agent: HTTPie/1.0.2

				username=chivt2&password=123456
				```
			-	Response
				```
				HTTP/1.1 201 CREATED
				Connection: close
				Content-Length: 57
				Content-Type: application/json
				Date: Wed, 06 Mar 2019 04:00:35 GMT
				Server: gunicorn/19.9.0

				{
					"message": "Account created successfully",
					"result": true
				}
				```
		-	URL xóa account **/account/delete**
			-	Methods **POST**
			-	Parameters
				-	username
				-	password
			-	Request
				```
				POST /account/delete HTTP/1.1
				Accept: */*
				Accept-Encoding: gzip, deflate
				Connection: keep-alive
				Content-Length: 32
				Content-Type: application/x-www-form-urlencoded; charset=utf-8
				Host: 127.0.0.1:5000
				User-Agent: HTTPie/1.0.2

				username=chivt22&password=123456
				```
			-	Response
				```
				HTTP/1.1 200 OK
				Connection: close
				Content-Length: 57
				Content-Type: application/json
				Date: Wed, 06 Mar 2019 04:26:52 GMT
				Server: gunicorn/19.9.0

				{
					"message": "Account deleted successfully",
					"result": true
				}
				```
		-	URL kiểm tra account **/account/check**
			-	Methods **POST**
			-	Parameters
				-	username
				-	password
			-	Request
				```
				POST /account/check HTTP/1.1
				Accept: */*
				Accept-Encoding: gzip, deflate
				Connection: keep-alive
				Content-Length: 31
				Content-Type: application/x-www-form-urlencoded; charset=utf-8
				Host: 127.0.0.1:5000
				User-Agent: HTTPie/1.0.2

				username=chivt2&password=123456
				```
			-	Response
				```
				HTTP/1.1 200 OK
				Connection: close
				Content-Length: 2
				Content-Type: text/html; charset=utf-8
				Date: Wed, 06 Mar 2019 04:26:00 GMT
				Server: gunicorn/19.9.0

				OK
				```
