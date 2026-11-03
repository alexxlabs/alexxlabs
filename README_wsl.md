https://learn.microsoft.com/en-us/windows/wsl/install

wsl --list --online
Ниже приведен список допустимых распределений, которые можно установить.
Установите с помощью команды wsl --install -d <Distro>.

NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Debian                          Debian GNU/Linux
kali-linux                      Kali Linux Rolling
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_10                Oracle Linux 8.10
OracleLinux_9_5                 Oracle Linux 9.5
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
openSUSE-Tumbleweed             openSUSE Tumbleweed

If the install process hangs at 0.0%, run:
wsl --install --web-download -d <DistroName>
to first download the distribution prior to installing.