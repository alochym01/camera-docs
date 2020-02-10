-   Nginx Javascript là một plugin được sử dụng giống như lua trên nginx
-   Nginx Javascript là đươc do chính nginx team phát triển

-   Mô hình chạy sẽ là như sau:

                        ---------------------                                   --------------
        CLient --------> | mqtt-authen-plugin | --proxy to mqtt-authen server--> | nginx:8010 | --proxy --> django authen API|
                        ---------------------                                   --------------

-   Repo cho nginx javascript

        [nginx-stable]
        name=nginx stable repo
        baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
        gpgcheck=1
        enabled=1
        gpgkey=https://nginx.org/keys/nginx_signing.key
        module_hotfixes=true

-   Cách cài đặt và cấu hình (đã cài đặt nginx rồi nha):

1.  `yum install nginx-module-njs`
2.  Copy file cấu hình mqtt-auth.conf với nội dung sau vào `/etc/nginx/conf.d/`

        proxy_cache_path /mnt/ssd/nginx levels=1:2 keys_zone=mqtt:10m inactive=24h max_size=50g;

        js_include conf.d/hello_world.js;

        log_format alochym '[$time_local] [Cache:$upstream_cache_status] $request_uri $request_body';

        server {
            listen       127.0.0.1:8010 default_server;
            server_name  _;

            access_log  /var/log/nginx/mqtt-access.log alochym;
            error_log /var/log/nginx/mqtt-error.log;

            location /queue/superuser {
                js_content tosuperuser;
            }
            location /queue/acl {
                proxy_cache_key $request_uri$request_body;
                proxy_ignore_headers Cache-Control Expires;
                proxy_cache mqtt;
                proxy_cache_methods POST;
                proxy_cache_valid 200 403 24h;
                proxy_pass http://127.0.0.1:8002/queue/acl;
            }
            location /queue/auth {
                proxy_cache_key $request_uri$request_body;
                proxy_ignore_headers Cache-Control Expires;
                proxy_cache mqtt;
                proxy_cache_methods POST;
                proxy_cache_valid 200 403 24h;
                proxy_pass http://127.0.0.1:8002/queue/auth;
            }
        }

3.  Create hello_world.js với nội dung sau vào `/etc/nginx/conf.d/`

        function tosuperuser(r) {
            //r.error(r.requestBody)
            var m_username = JSON.parse(r.requestBody)['username'];

            var regex = RegExp('Alochym*');
            //r.error("user comming to: " + m_username);
            if (regex.test(m_username)) {
                //r.error("sys user comming to: " + m_username);
                r.return(200,"OK");
            }else{
                //r.error("user binh thuong vao superuser: " + m_username);
                r.return(403,"denied");
            }
        }

4.  `systemctl restart nginx`
