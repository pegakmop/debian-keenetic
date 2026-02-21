установка ndmq на debian aarch64

``` 
cd /root
wget https://raw.githubusercontent.com/The-BB/debian-keenetic/refs/heads/master/EOL/ndmq-aarch64_bullseye.tgz
tar -xzf ndmq-aarch64_bullseye.tgz -C /
ls -l /usr/local/bin/ndmq
ls -l /usr/local/lib/libndm.so
ldconfig
ldd /usr/local/bin/ndmq
ndmq -v
ndmq -help
``` 
