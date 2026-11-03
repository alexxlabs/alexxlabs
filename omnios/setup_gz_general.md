### Creating Repos
#
# https://github.com/omniosorg/omnios-wiki/blob/master/CreatingRepos.md#creating-repos
# https://habr.com/ru/articles/1067250/

# https://github.com/omniosorg/omnios-wiki/blob/master/GeneralAdministration.md

### Upgrading With Non-Global Zones
#
# https://github.com/omniosorg/omnios-wiki/blob/master/GeneralAdministration.md#upgrading-with-non-global-zones

### LX Branded Zones
#
# https://github.com/omniosorg/omnios-wiki/blob/master/LXZones.md#lx-branded-zones

### ===================== manual clear invalid zone ===============================

# zadm delete <zone>
# Error: Found 2 top-level filesystems
Принудительное удаление конфигурации зоны
Утилита zonecfg работает на более низком уровне, чем zadm, и позволит обойти эту ошибку:

zonecfg -z <zone-name> delete -F

Очистка хранилища в ZFS
После того как конфигурация удалена, zadm больше не будет контролировать эту зону.
Теперь вам нужно вручную уничтожить файловые системы этой зоны в ZFS, которые и вызвали ошибку:
Найдите проблемные датасеты:

zfs list -r rpool/zones/<имя-зоны>

Принудительно удалите их:

zfs destroy -rf rpool/zones/<имя-зоны>
### ===================== /manual clear invalid zone ===============================