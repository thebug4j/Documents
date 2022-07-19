# Nginx: 配置SSL证书启动报错



1、报错信息

```
nginx: [emerg] SSL_CTX_use_certificate("~/etc/client.crt") failed (SSL: error:140AB18E:SSL routines:SSL_CTX_use_certificate:ca md too weak)
```



2、错误原因

```
证书加密强度过低
```



3、解决

```
方案一：证书重新生成
```

```
方案二：降级openssl 版本

1、服务器上执行下列命令查看版本，（出现问题的版本是 OpenSSL 1.1.1n  15 Mar 2022）
openssl version 
2、降级

	
```

