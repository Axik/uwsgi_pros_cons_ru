### How was tested
Prerequisite requirements:
 - Python
 - [wrk](https://github.com/wg/wrk) as a load tool
 - [nginx](https://nginx.org/) as a http reverse proxy.

```
$ mkvirtualenv load_test
$ pip install -r requirements.txt
```
My cpu info
```
$ sysctl -n machdep.cpu.brand_string
Intel(R) Core(TM) i7-4870HQ CPU @ 2.50GHz
```
Nginx config for uwsgi-http mode and gunicorn.
```

worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

         location / {
            proxy_pass http://127.0.0.1:5000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

    }
}
```
For nginx-uwsgi protocol it's slightly changes. You can read how to setup it [here](!http://uwsgi-docs.readthedocs.io/en/latest/Nginx.html)

Run server with gunicorn easy:
```
$ gunicorn sample:api -w 8 -p 5000 -b 127.0.0.1:5000
```
Make load burst in new terminal tab.

Run server with uwsgi:
```
$ uwsgi --wsgi-file sample.py --callable api --http :5000 -p 8 -H ~/.virtualenvs/uwsgi_tests
```
