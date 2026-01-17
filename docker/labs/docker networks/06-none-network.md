## None Network

### Step 1: Run none network
```sh
bash-5.1$ docker run -it --network none busybox
```

### Step 2: Test networking
```sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever

/ # ping google.com
ping: bad address 'google.com'
```

**Note:** Used for batch jobs, security workloads
