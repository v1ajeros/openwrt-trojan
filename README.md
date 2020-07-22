openwrt-trojan
==============

Usage
-----

1. copy these two folders to <openwrt-source-tree>/package.

2. install feeds from openwrt official package repository.

    ./scripts/feeds update -a
    ./scripts/feeds install -a

3. use 'make menuconfig' to select trojan package

4. the buildroot generate trojan binary linked to our openssl.
   You may use 'make package/trojan/{clean,compile} V=99' or
   whatever you like.

5. edit '/etc/config/trojan' file to enable it.
   The init script is disabled by default to avoid startup
   before configuration.
   
6. edit '/etc/trojan.json' file. 
<pre><code>{
    "run_type": "nat",
    "local_addr": "127.0.0.1",
    "local_port": 12345,
    "remote_addr": "example.com",
    "remote_port": 443,
    "password": [
        "password1"
    ],
    "log_level": 1,
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:AES128-SHA:AES256-SHA:DES-CBC3-SHA",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "sni": "",
        "alpn": [
            "h2",
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "curves": ""
    },
    "tcp": {
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    }
}</code></pre>

7. edit '/etc/firewall.user' for transparent proxy
<pre><code># Create new chain
iptables -t nat -N TROJAN
iptables -t mangle -N TROJAN

# Ignore your trojan server's addresses
# It's very IMPORTANT, just be careful.
iptables -t nat -A TROJAN -d <server ip> -j RETURN

# Ignore LANs and any other addresses you'd like to bypass the proxy
# See Wikipedia and RFC5735 for full list of reserved networks.
# See ashi009/bestroutetb for a highly optimized CHN route list.
iptables -t nat -A TROJAN -d 0.0.0.0/8 -j RETURN
iptables -t nat -A TROJAN -d 10.0.0.0/8 -j RETURN
iptables -t nat -A TROJAN -d 127.0.0.0/8 -j RETURN
iptables -t nat -A TROJAN -d 169.254.0.0/16 -j RETURN
iptables -t nat -A TROJAN -d 172.16.0.0/12 -j RETURN
iptables -t nat -A TROJAN -d 192.168.0.0/16 -j RETURN
iptables -t nat -A TROJAN -d 224.0.0.0/4 -j RETURN
iptables -t nat -A TROJAN -d 240.0.0.0/4 -j RETURN

# Anything else should be redirected to trojan's local port
iptables -t nat -A TROJAN -p tcp -j REDIRECT --to-ports <server port>

# Add any UDP rules
ip route add local default dev lo table 100
ip rule add fwmark 1 lookup 100
iptables -t mangle -A TROJAN -d 0.0.0.0/8 -j RETURN
iptables -t mangle -A TROJAN -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A TROJAN -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A TROJAN -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A TROJAN -d 172.16.0.0/12 -j RETURN
iptables -t mangle -A TROJAN -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A TROJAN -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A TROJAN -d 240.0.0.0/4 -j RETURN
iptables -t mangle -A TROJAN -p udp -j TPROXY --on-port <server port> --tproxy-mark 0x01/0x01

# Apply the rules
iptables -t nat -A PREROUTING -p tcp -j TROJAN
iptables -t mangle -A PREROUTING -j TROJAN
</code></pre>
FAQ
---

Q: May I use openssl from openwrt?
A: As long as you don't need cutting-edge features, e.g. TLS 1.3.
   BTW, the Makefile doesn't depend on official openssl package.
