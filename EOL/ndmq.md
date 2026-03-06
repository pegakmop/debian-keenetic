установка ndmq на debian aarch64

``` 
apt install wget -y
cd /root
wget https://raw.githubusercontent.com/The-BB/debian-keenetic/refs/heads/master/EOL/ndmq-aarch64_bullseye.tgz
tar -xzf ndmq-aarch64_bullseye.tgz -C /
ls -l /usr/local/bin/ndmq
ls -l /usr/local/lib/libndm.so
ldconfig
ldd /usr/local/bin/ndmq
ndmq -v
ndmq -help
cat > /usr/local/bin/ndmc << 'EOF'
#!/bin/sh

# /usr/local/bin/ndmc
# chmod +x /usr/local/bin/ndmc

if [ "$1" = "-c" ]; then
    shift
    ndmq -p "$*" -x | sed \
        -e 's/<response>//' \
        -e 's#</response>##' \
        -e 's#<prompt>.*</prompt>##'
else
    ndmq -x
fi
EOF
chmod +x /usr/local/bin/ndmc
``` 
