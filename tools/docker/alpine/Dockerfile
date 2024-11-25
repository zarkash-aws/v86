FROM docker.io/i386/alpine:3.20.0

ENV KERNEL=lts
ENV ADDPKGS=nodejs

RUN apk add openrc alpine-base agetty alpine-conf $ADDPKGS gcompat libc6-compat valkey valkey-cli

RUN if [ "$KERNEL" == "lts" ]; then \
    apk add linux-lts \
            linux-firmware-none \
            linux-firmware-sb16; \
else \
    apk add linux-$KERNEL; \
fi

RUN sed -i 's/getty 38400 tty1/agetty --autologin root tty1 linux/' /etc/inittab
RUN echo 'ttyS0::respawn:/sbin/agetty -l/usr/bin/start.sh --autologin root -s ttyS0 115200 vt100' >> /etc/inittab
RUN echo "root:" | chpasswd
COPY start.sh /usr/bin/start.sh
RUN chmod 755 /usr/bin/start.sh
COPY networking.sh /usr/bin/networking.sh
RUN chmod 755 /usr/bin/networking.sh
RUN setup-hostname localhost

# Adding networking.sh script (works only on lts kernel yet)
#RUN if [ "$KERNEL" == "lts" ]; then \
#    echo -e "rmmod ne2k-pci && modprobe ne2k-pci\nhwclock -s\nsetup-interfaces -a -r" > /root/networking.sh && \
#    chmod +x /root/networking.sh; \
#fi

RUN echo 'console.log("Hello, world!");' > /root/hello.js

# https://wiki.alpinelinux.org/wiki/Alpine_Linux_in_a_chroot#Preparing_init_services
RUN for i in devfs dmesg mdev hwdrivers; do rc-update add $i sysinit; done
RUN for i in hwclock modules sysctl hostname syslog bootmisc; do rc-update add $i boot; done
RUN rc-update add killprocs shutdown

# Generate initramfs with 9p modules
RUN mkinitfs -F "ata base ide scsi virtio ext4 9p" $(cat /usr/share/kernel/$KERNEL/kernel.release)
