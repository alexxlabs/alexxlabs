# inspired by:	https://blog.mark99.ru/generacziya-liczenzii-supermicro-x8-x9-x10-x11/
#				https://peterkleissner.com/2018/05/27/reverse-engineering-supermicro-ipmi/

Для выполнения обновления BIOS и управления RAID контроллером из IPMI Supermicro материнская плата
требует активацию лицензионным ключом. На новых платах она требует файл лицензии и эта инструкция не работает.
Для старых лицензионный ключ можно сгенерировать

Лицензионный ключ можно сгенерировать, имея MAC адрес BMC, который указан на главной странице
при входе в панель управления IPMI.
для моей материнки: BMC MAC Address: 0c:c4:7a:af:03:d0

Генерация ключа происходит следующей командой в Bash:

echo -n '0cc47aaf03d0' | xxd -r -p | openssl dgst -sha1 -mac HMAC -macopt hexkey:8544E3B47ECA58F9583043F8 | awk '{print $2}' | cut -c 1-24

Для MAC адреса BMC: 0c:c4:7a:af:03:d0
будет сгенерирован код лицензии: 57a40358432b69aefd003459
57A4-0358-432B-69AE-FD00-3459

# Update 6/8/2021: Ben provided the following improved script:

#!/bin/bash
read -p "IPMI MAC: " macaddr
echo -n $macaddr | xxd -r -p | openssl dgst -sha1 -mac HMAC -macopt hexkey:8544E3B47ECA58F9583043F8 | awk '{print $2}' | cut -c 1-24 | sed 's/./&-/4;s/./&-/9;s/./&-/14;s/./&-/19;s/./&-/24;'

-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------
-------------------------------------- another way, more functionality ------------------------------------------
-----------------------------------------------------------------------------------------------------------------

https://www.supermicro.com/wdl/utility/IPMIView/Windows/

!!! if you need HTML5 console inside this tool, you need to activate SFT-DCMS-SINGLE licence

SFT-DCMS-Single:	4d75-d64b-8348-5993-5c2a-6763 [???]
-----------------
SFT-DCMS-SINGLE is Supermicro’s Data Center Management Suite license that enables server node
				to take full advantage of Supermicro Management Software and Utilities features.

Benefits:

System Lockdown: System lockdown helps in preventing the system from unintentional or malicious changes. This feature can help in protecting Lockdown mode is applicable to both configuration and firmware updates. When system lockdown is turned on, all system configuration changes including firmware updates will be prevented and user will be notified accordingly.
Redfish Support:The Redfish Scalable Platforms Management API ("Redfish") is a new interface that uses RESTful interface semantics to access data defined in a model format to perform out-of-band systems management. It is suitable for a wide range of servers, from stand-alone to rack mount and blade environments, but scales equally well for large scale cloud environments. Utilizing JSON (JavaScript Object Notation) data format, allows many types of parameters to be available such that it enables scalability, human readability, and flexibility for most programming environments by easily interpreting payload.
HTML5 Virtual Media Support: Supermicro BMC provides HTML5-based remote presence (iKVM) for user management remotely far away from the place. With the SFT-DCMS-SINGLE license key enabled, user is able to configure the ISO/IMA images mount thru virtual media feature. It also supports the configuration thru Redfish.

this is how to do this:

use this tool: https://github.com/zsrv/supermicro-product-key
use 06_vm_torr VM, inside this VM golang is installed:
-------
$ cd 06_vm_torr
$ dz vm zlogin
$ cd /root
$ go install github.com/zsrv/supermicro-product-key@latest
...
$ go/bin/supermicro-product-key nonjson encode --sku SFT-DCMS-SINGLE 0cc47aaf03d0

AAYAAAAAAAAAAAAAAAAAAG3PWTHWUDTVdKI1MF/fSiwdW8i+2YtmvBhrTIRzd+jn1bd1XzWXEBOykmNy+u6PAs5EzxbJ3Iy08AQYWukZn42NuixcX6PMfg0p6sQf8kTnhVlFF5JSXKVZUpCIpwq37epq2D145Pb+MWnra/GXW4nefWt3YoKjTJ4wnxTaVQkEh5J5n+jFHC7m2mIA2MtoVFhXWdlTrvWEXWvq7xnN8XKq0gYC9Hp8JR01HPRiiuc71h6D0veSlC3VzKEbDJWT6JiWpG+KDpg/JBgi3TjurrL/9dl6TJ1ajpsXzj233mmY70sTMmUCLKKZi/qi3zi7lA==

Take that string and install it with Supermicro SUM like that
The standalone distributions of SUM 2.4.0 can no longer be downloaded directly from the Supermicro website,
but it is available inside the BIOS and BMC firmware update package for the X11DPG-HGX2.
Download page: https://www.supermicro.com/en/support/resources/downloadcenter/firmware/MBD-X11DPG-HGX2/BIOS
Direct link: https://www.supermicro.com/Bios/softfiles/11407/X11DPG-HGX2_3_3_HG11000V_SUM240.zip

.\sum.exe -i 10.2.0.16 -u alexxlabs -p <put_pass_here> -c ActivateProductKey --key AAYAAAAAAAAAAAAAAAAAAG3PWTHWUDTVdKI1MF/fSiwdW8i+2YtmvBhrTIRzd+jn1bd1XzWXEBOykmNy+u6PAs5EzxbJ3Iy08AQYWukZn42NuixcX6PMfg0p6sQf8kTnhVlFF5JSXKVZUpCIpwq37epq2D145Pb+MWnra/GXW4nefWt3YoKjTJ4wnxTaVQkEh5J5n+jFHC7m2mIA2MtoVFhXWdlTrvWEXWvq7xnN8XKq0gYC9Hp8JR01HPRiiuc71h6D0veSlC3VzKEbDJWT6JiWpG+KDpg/JBgi3TjurrL/9dl6TJ1ajpsXzj233mmY70sTMmUCLKKZi/qi3zi7lA==

After that you should see:

	Supermicro Update Manager (for UEFI BIOS) 2.4.0 (2019/12/06) (x86_64)
	Copyright(C) 2013-2019 Super Micro Computer, Inc. All rights reserved.
	Node product key (SFT-DCMS-Single) is activated!!

