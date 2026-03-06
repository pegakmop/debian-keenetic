для запуска и работы дебиана внутри ентвара
```
# Создаем директорию и распаковываем Debian
mkdir -p /opt/debian
tar -xzf ~/debian-trixie-13.0-aarch64.tar.gz -C /opt/debian

# Создаем скрипт для входа в chroot
cat > /opt/bin/debian << 'EOF'
#!/bin/sh
chroot /opt/debian/debian /bin/bash -l -c "cd /root && exec /bin/bash -l"
EOF
chmod +x /opt/bin/debian

# Создаем скрипт инициализации сервисов
cat > /opt/etc/init.d/S99debian << 'EOF'
#!/bin/sh

PATH=/opt/bin:/opt/sbin:/sbin:/bin:/usr/sbin:/usr/bin

# Debian folder
CHROOT_DIR=/opt/debian/debian

# Some folder outside of sandbox, will be mounted to /mnt folder in Debian
# Leave commented if not needed
#EXT_DIR=/media

CHROOT_SERVICES_LIST=$CHROOT_DIR/chroot-services.list
if [ ! -e "$CHROOT_SERVICES_LIST" ]; then
	echo "Please, define Debian services to start in $CHROOT_SERVICES_LIST first!"
	echo 'One service per line. Hint: this is a script names from Debian /etc/init.d/'
	exit 1
fi

MountedDirCount="$(mount | grep $CHROOT_DIR | wc -l)"

start() {
	if [ $MountedDirCount -gt 0 ]; then
		logger 'Debian services seems to be already started, exiting...'
		exit 1
	fi
	logger 'Starting Debian services...'
	for dir in dev dev/pts proc sys opt/etc; do
		mount -o bind /$dir $CHROOT_DIR/$dir
	done
	[ -z "$EXT_DIR" ] || mount -o bind $EXT_DIR $CHROOT_DIR/media
	for item in $(cat $CHROOT_SERVICES_LIST); do
		PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/sbin \
		LC_ALL=C \
		LANGUAGE=C \
		LANG=C \
		chroot $CHROOT_DIR /etc/init.d/$item start
	done
}

stop() {
	if [ $MountedDirCount -eq 0 ]; then
		logger 'Debian services seems to be already stopped, exiting...'
		exit 1
	fi
	logger 'Stopping Debian services...'
	for item in $(cat $CHROOT_SERVICES_LIST); do
		PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/sbin \
		LC_ALL=C \
		LANGUAGE=C \
		LANG=C \
		chroot $CHROOT_DIR /etc/init.d/$item stop
	done
	umount $CHROOT_DIR/dev/pts
	mount | grep $CHROOT_DIR | awk "{print \$3}" | xargs umount
}

status() {
	if [ $MountedDirCount -gt 0 ]; then
		echo 'Debian services is running'
	else
		echo 'Debian services is stopped'
	fi
}

case "$1" in
	start)
		start
	;;
	stop)
		stop
	;;
	restart)
		stop
		sleep 5
		start
	;;
	status)
		status
	;;
	*)
		echo "Usage: $0 (start|stop|restart|status)"
		exit 1
	;;
esac

echo 'Done.'
exit 0
EOF

chmod +x /opt/etc/init.d/S99debian
# настраиваем подмену ndmq
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

# Создаем файл со списком сервисов (пример)
cat > /opt/debian/debian/chroot-services.list << 'EOF'
# Примеры сервисов (раскомментируйте нужные)
# ssh
# cron
# nginx
EOF

echo "Установка завершена!"
echo "Для входа в Debian используйте: debian"
echo "Для управления сервисами: /opt/etc/init.d/S99debian {start|stop|restart|status}"
```
запускаем дебиан и входим
```
/opt/etc/init.d/S99debian start && debian
```
выход командой
```
exit
``` 
