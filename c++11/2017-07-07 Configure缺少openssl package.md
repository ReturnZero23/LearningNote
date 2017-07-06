# libjwt 编译过程
1. 编译过程中报找不到openssl
```
➜  libjwt git:(master) ./configure    
.....
.....
checking for OPENSSL... no
configure: error: Package requirements (openssl >= 0.9.8) were not met:

No package 'openssl' found

Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.

Alternatively, you may set the environment variables OPENSSL_CFLAGS
and OPENSSL_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.
```
2. 但是终端是可以调用openssl的：
```
➜  ~ openssl version  
OpenSSL 1.0.2g  1 Mar 2016
```
3. 解决方案：需要安装openssl的开发包，安装
- Red Hat, Fedora, CentOS - openssl-devel
- Debian, Ubuntu - libssl-dev
- Arch - openssl
参考[superuser.com](https://superuser.com/questions/371901/openssl-missing-during-configure-how-to-fix)