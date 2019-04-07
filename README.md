# Slowloris defence example

### How to run

```
docker-compose up -d --build
```

### What was done to mitigate the attack

1. Limiting number of connections from single IP address: added following config to nginx `default.conf` file
    ```
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    
    server {
        ...
        location / {
            limit_conn addr 10;
            ...
        }
        ...
    }
    ```
2. Closing slow connections: added following config to nginx `default.conf` file
    ```
    server {
        ...
        
        client_body_timeout 5s;
        client_header_timeout 5s;
    
        ...
    }
    ```
3. Increasing NGINX worker connections limit: added following config to nginx `nginx.conf` file
    ```
    events {
        worker_connections  100000;
    }
    ```
4. Increasing user’s open file limit: added following config  to nginx container in `docker-compose.yml` file 
    ```yaml
    ulimits:
      nofile:
        soft: "100000"
        hard: "100000"
    ```
5. Increasing NGINX’s worker number of open files limit: added following config to nginx `nginx.conf` file
    ```
    worker_rlimit_nofile 102400;
    ```
    
### Sources used
1. [Nginx blog article](https://www.nginx.com/blog/mitigating-ddos-attacks-with-nginx-and-nginx-plus/)
2. [HEXADIX blog article](https://hexadix.com/slowloris-dos-attack-mitigation-nginx-web-server/)

Slowloris implementation was forked from https://github.com/valyala/goloris and POST requests were replaced with GET in 
order to make requests to Nginx static page (Nginx forbids POST requests to static pages)
