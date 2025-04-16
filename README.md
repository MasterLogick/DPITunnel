<div align="center">
<img src="assets/logo.webp" alt="DPI Tunnel logo" width="200">
<br><h1>DPI Tunnel for Linux</h1>
Free, simple and serverless solution against censorship for Linux PCs and routers

<a href="https://github.com/txtsd/DPITunnel/blob/main/LICENSE"><img src="https://img.shields.io/github/license/txtsd/DPITunnel?style=flat-square" alt="License"/></a>
<a href="https://github.com/txtsd/DPITunnel/releases/latest"><img src="https://img.shields.io/github/v/release/txtsd/DPITunnel?style=flat-square" alt="Latest release"/></a>
<a href="https://github.com/txtsd/DPITunnel/releases"><img src="https://img.shields.io/github/downloads/txtsd/DPITunnel/total?style=flat-square" alt="Downloads"/></a>
</div>

### What is it

DPI Tunnel is a proxy server, that allows you to bypass censorship

It is NOT a VPN and will NOT change your IP

DPI Tunnel uses desync attacks to fool DPI filters

RUN IT AS ROOT

### Features

* Bypass many restrictions: blocked or throttled resources
* Create profiles for different ISPs and automatically change them when the connection switches
* Easily autoconfigure for your ISP
* Has HTTP and transparent proxy modes

## Configuring

### For most of the ISPs, one of the these 2 profiles will be enough

```shell
--ca-bundle-path=<path_to_cabundle> --desync-attacks=fake,disorder_fake --split-position=2 --auto-ttl=1-4-10 --min-ttl=3 --doh --doh-server=https://dns.google/dns-query --wsize=1 --wsfactor=6
```

```shell
--ca-bundle-path=<path_to_cabundle> --desync-attacks=fake,disorder_fake --split-position=2 --wrong-seq --doh --doh-server=https://dns.google/dns-query --wsize=1 --wsfactor=6
```

*CA Bundle is a file that contains root and intermediate SSL certificates. Required for DoH and autoconfig to work. You
can get it for example from the [curl](https://curl.se/ca/cacert.pem) website*

#### For other ISPs, `--auto` will automatically find proper settings

## Running

### HTTP mode (default)

This mode is good for PC or any other device which will only use the proxy for itself.

Run executable with options either from autoconfig or from one of the suggested profiles. The program will tell IP and
port on which the proxy server is running. 0.0.0.0 IP means any of IPs this machine has.

Set this proxy in browser or system settings

### Transparent mode

This mode is good for router which will use the proxy for the entire local network.

Run executable with `--mode transparent` and append options either from autoconfig or from one of the suggested
profiles. The program will tell IP and port on which the proxy server is running. 0.0.0.0 IP means any of IPs this
machine has.

#### If proxy running on router

##### 1. Enable IP forwarding

```bash
sysctl -w net.ipv4.ip_forward=1
```

##### 2. Disable ICMP redirects

```bash
sysctl -w net.ipv4.conf.all.send_redirects=0
```

##### 3. Enter something like the following ```iptables``` rules

```bash
iptables -t nat -A PREROUTING -i <iface> -p tcp --dport 80 -j REDIRECT --to-port <proxy_port>
iptables -t nat -A PREROUTING -i <iface> -p tcp --dport 443 -j REDIRECT --to-port <proxy_port>
```

#### If proxy running on machine in local network (Raspberry PI for example)

##### 1. On router

```bash
iptables -t mangle -A PREROUTING -j ACCEPT -p tcp -m multiport --dports 80,443 -s <proxy_machine_ip>
iptables -t mangle -A PREROUTING -j MARK --set-mark 3 -p tcp -m multiport --dports 80,443
ip rule add fwmark 3 table 2
ip route add default via <proxy_machine_ip> dev <iface> table 2
```

##### 2. On proxy machine

1. Enable IP forwarding

```bash
sysctl -w net.ipv4.ip_forward=1
```

2. Disable ICMP redirects

```bash
sysctl -w net.ipv4.conf.all.send_redirects=0
```

3. Enter something like the following ```iptables``` rules:

```bash
iptables -t nat -A PREROUTING -i <iface> -p tcp --dport 80 -j REDIRECT --to-port <proxy_port>
iptables -t nat -A PREROUTING -i <iface> -p tcp --dport 443 -j REDIRECT --to-port <proxy_port>
```

## Docker

You could run dpitunnel as docker container

1. Build image:
```bash
docker build -t dpitunnel .
```

2. Start proxy
```bash
docker run -it -p 8080:8080 dpitunnel -doh -doh-server https://dns.google/dns-query -ttl 1 -ca-bundle-path /etc/ssl/certs/ca-certificates.crt -desync-attacks disorder_fake
```

Image is based on alpine, that provides ca certs bundle on path `/etc/ssl/certs/ca-certificates.crt`. You can use it in `ca-bundle-path` option.

## Links

* [Telegram chat](https://t.me/DPITunnelOFFICIAL)
* [Android Version](https://github.com/nomoresat/DPITunnel-android)
* [4PDA](https://4pda.to/forum/index.php?showtopic=1043778)

## Thanks

* [ValdikSS (GoodbyeDPI)](https://github.com/ValdikSS/GoodbyeDPI)
* [nomoresat (DPITunnel-cli)](https://github.com/nomoresat/DPITunnel-cli)

## Dependencies

* [RawSocket](https://github.com/chkpk/RawSocket)
* [cpp-httplib](https://github.com/yhirose/cpp-httplib)
* [dnslib](https://github.com/mnezerka/dnslib)
* [libnl](https://www.infradead.org/~tgr/libnl)
