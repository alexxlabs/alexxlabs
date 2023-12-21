The tmpfs can also be temporarily resized without the need to reboot,
for example when a large compile job needs to run soon. In this case, run:

mount -o remount,size=4G,noatime /

### OpenIPC
python burn --chip hi3516ev200 --file=u-boot-gk7205v300-universal.bin -p COM2 --break 
&& C:\portable\putty\PUTTY.EXE -serial COM4 -sercfg 115200,8,n,1,N

https://github.com/OpenIPC/burn
https://github.com/OpenIPC/firmware/releases
