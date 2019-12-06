## FTP Server
#### Component
-	Envoy Server
-	FTP server
	-	pam_url plugin
	-	pam_script plugin
-	Nginx Server
-	Flask framework
	-	sqlite database
#### How it works
-	Envoy server
	-	acts as FTP proxy forward all FTP connection from clients to FTP server
	-	using tcp_proxy filter chain
	-	using prometheus for monitoring
-	FTP server
	-	works as FTP server
	-	using pam_url plugins to send(POST method) authenticated FTP request login to flask API for authentication
	-	using pam_script plugin to create a folder for client after successfully authentication
	- vsftpd has a definition virtual users, what is virtual users
		-	The user is not working on the OS system level
		-	The user is only meaning with ftp account
-	Nginx server
	-	acts as web reverse proxy for caching authenticatons for flask API
-	Flask framework
	-	acts as API web server
	-	Flask routes
		```ss
			Endpoint        Methods  Rule
			--------------  -------  -----------------------
			account.check   POST     /account/check
			account.create  POST     /account/create
			account.delete  POST     /account/delete
			account.index   GET      /account/
			static          GET      /static/<path:filename>