apk add gmake cmake
alsa-lib alsa-lib-devel linux-headers
alsa-utils alsa-utils-locale alsa-utils-man
i2c-tools i2c-tools-devel
apk add llvm-devel lld-devel

instead of simple make, do:
	make CC=clang

# Скомпилируйте файл оверлея для вашей платы. 
cd /data/seeed-voicecard
dtc -@ -I dts -O dtb -o /boot/overlays/seeed-4mic-voicecard.dtbo ./seeed-4mic-voicecard-overlay.dts

# compile and install dtbos from source so the card names are correct
for dts in seeed-2mic-voicecard-overlay.dts seeed-4mic-voicecard-overlay.dts seeed-8mic-voicecard-overlay.dts; do
  dtbo="${dts%-overlay.dts}.dtbo"
  dtc -@ -I dts -O dtb -o "$dtbo" "$dts" 2>/dev/null || {
    echo "Warning: dtc failed for $dts, using pre-built $dtbo"
  }
done

# TODO: review script ./seeed_voicecard ( which is doing all setup)
# and rewrite it inside /etc/rc.local
#
# for now, inside /etc/rc.local we do:
modprobe i2c-dev
modprobe snd_soc_ac108
modprobe snd_soc_wm8960
modprobe snd_soc_seeed_voicecard

# to compress modules:
cd /data/seeed-voicecard
zstd -c ./snd-soc-ac108.ko > /lib/modules/6.18.59-0-rpi/kernel/sound/soc/codecs/snd-soc-ac108.ko.zst
zstd -c ./snd-soc-wm8960.ko > /lib/modules/6.18.59-0-rpi/kernel/sound/soc/codecs/snd-soc-wm8960.ko.zst
zstd -c ./snd-soc-seeed-voicecard.ko > /lib/modules/6.18.59-0-rpi/kernel/sound/soc/codecs/snd-soc-seeed-voicecard.ko.zst

use 'depmod' to re-generate the module dependencies