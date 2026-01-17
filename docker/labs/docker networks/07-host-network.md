## Host Network (No Isolation)

### Step 1: Run nginx on host network
```sh
bash-5.1$ docker run -d --network host nginx
98708b650334c6e792fbf755b9ef52724003fe5433691358a0d706dd02461d74

bash-5.1$ docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
3988f947a319   app-net        bridge    local
118f65fe21ff   bridge         bridge    local
e5203f00748b   host           host      local
df26996a588f   isolated-net   bridge    local
82714d432fcc   none           null      local
```
### Step 2: Access directly
```sh
bash-5.1$ curl http://localhost
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
bash-5.1$ 
```

#### NOTE:
  - Container shares host network
  - No port mapping
  - Linux only
  - Risky but fast
