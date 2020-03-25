Nginx Image Filter Plugin là một plugin được sử dụng để trực tiếp xử lí ảnh khi client muốn lấy ảnh trên nginx (ở đây chúng ta sẽ là resize lại ảnh )

Mô hình chạy sẽ là như sau:

```text
                 ------------             	              --------------
CLient --------> | nginx:80 | --proxy to internal resize--> | nginx:7070 | <- Internal resize
                 ------------             	              --------------
|_____Cache result <---|  |____return resized data if valid <------|
```

Cách cài đặt và cấu hình (đã cài đặt nginx rồi nha):

1.  `yum install nginx-mod-http-image-filter`
2.  Copy file cấu hình imageresize.conf với nội dung sau vào `/etc/nginx/conf.d/`

        server {
            listen 7070;

            root /home/nginx/thumbnails/;
            location ~ /resize/([\d-]+)x([\d-]+)/(.*) {

                proxy_pass                 http://$server_addr:7070/$3;
                proxy_set_header Host      $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_buffer_size 512k;
                proxy_buffers 16 512k;

                image_filter                resize $1 $2;
                image_filter_jpeg_quality   80;
                image_filter_buffer         10M;
            }

            location ~ (.*) {
                try_files                   /$1 @img;
            }

            location @img {
                proxy_pass http://$server_addr/$uri;
            }

        }

        proxy_cache_path /tmp/nginx-images-cache/ levels=1:2 keys_zone=images:10m inactive=24h max_size=100m;

        server {
            # Public-facing cache server.
            listen 80;

            # Only serve widths of 768 or 1920 so we can cache effectively.
            location ~ "^/thumbnails/(?<image>.+)$" {
                # Proxy to internal image resizing server.
                proxy_pass http://localhost:7070/resize/100x56/$image;
                proxy_cache images;
                proxy_cache_valid 200 24h;
            }
        }

3.  `systemctl restart nginx`
