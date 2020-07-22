openwrt-trojan
==============

Usage
---

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
<pre><code>
{
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
}
</code></pre>

FAQ
---

Q: May I use openssl from openwrt?
A: As long as you don't need cutting-edge features, e.g. TLS 1.3.
   BTW, the Makefile doesn't depend on official openssl package.
