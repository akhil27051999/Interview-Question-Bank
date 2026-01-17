## Port Mapping (Host ↔ Container)

### Step 1: Expose nginx
```sh
bash-5.1$ docker run -d \
  --name web-pub \
  --network app-net \
  -p 8080:80 \
  nginx
2948619f33021eabe485fb6dde9a6a673a4bce1d72c9da5f738c3ef04542425f

bash-5.1$ docker ps | grep web-pub
2948619f3302   nginx     "/docker-entrypoint.…"   15 seconds ago   Up 14 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   web-pub
```

### Step 2: Test from host
```sh
bash-5.1$ curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### Note:
  - `-p` is host → container
  - Containers do NOT need ports mapped to talk internally
