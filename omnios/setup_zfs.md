OmniOS как основа 'домашнего облака': https://habr.com/ru/articles/1010372/

### Ограничение размера Adaptive Replacement Cache (ARC).
По-умолчанию размер ARC вычисляется автоматически,
и в результате такового может составить от 50% до 100% оперативной памяти.
Самый простой способ - добавить в файл
	/etc/system.d/_omnios:zfs:defaults
строку:

echo "# Limits ARC to 16GB, adjust as needed" > /etc/system.d/_omnios:zfs:defaults
??? echo "set zfs:zfs_arc_max = 0x400000000" > /etc/system.d/_omnios:zfs:defaults
echo "set zfs:zfs_arc_max = 68719476736" > /etc/system.d/_omnios:zfs:defaults

Где 68719476736 = 16 * 1024^3, т.е 16 гигабайт.
Для домашнего файлового сервера, учитывая интенсивность использования,
это очень большая величина. Можно при необходимости сократить до 4-х или до 2-х гигабайт.
