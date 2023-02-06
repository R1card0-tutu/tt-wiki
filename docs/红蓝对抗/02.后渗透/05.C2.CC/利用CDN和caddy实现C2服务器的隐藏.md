# 0x00 技术路线

cdn -> caddy -> pupy / cs

# 0x01 caddy配置

下载

```
https://github.com/caddyserver/caddy/releases
tar -zxvf caddy_2.5.2_linux_amd64.tar.gz
```

添加到PATH

```
mv caddy /usr/local/bin/
```

测试是否安装成功

```
caddy run 
curl localhost:2019/config/
```

# 0x02 caddy 配置文件

caddy的配置文件

```
{
#     admin off
    auto_https disable_redirects
}

(handle_error) {
    @error status 404 502 503
    handle_response @error {
        respond /404.html 404
    }
}

(file_server_common) {
    encode gzip
    file_server
    header -Server
}

http://:60080 {
    log {
        output file /home/caddy/caddy.log
    }
}

# CS服务器
http://e80f42a521.static.qianxin.com:60080 {
    root * /home/caddy/webroot

    import file_server_common

    reverse_proxy https://localhost:60002 {
        transport http {
            tls_insecure_skip_verify
        }
        import handle_error
        header_up Host {upstream_hostport}
        header_up X-Forwarded-Host {host}
        header_up X-Forwarded-Port {port}
    }

    handle_errors {
        rewrite * /404.html
        import file_server_common
    }
}
```

# 0x03 配置cdn

常规配置，需要注意的点

- 取消缓存
- 源站地址设置的时候记得指定port
- ![image-20220722233430603](../../../../../../Library/Application Support/typora-user-images/image-20220722233430603.png)

- 回源HOST随便设置（cdn就是一个流量转发的，只要保证cdn发出来的流量能到caddy就行）

# 0x04 CS需要配置的点

- Profile修改X-Forward-For，使能够获取到真实上线IP
- listener配置，bindport是真实监听的端口

# 0x05 pupy配置

安装

```
git clone --recursive https://github.com/n1nj4sec/pupy

```

