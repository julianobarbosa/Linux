# Package Install
```console
yum install bash-completion bash-completion-extras mlocate wget tmux
```
# PowerShell
## Register the Microsoft RedHat repository
```console
curl https://packages.microsoft.com/config/rhel/7/prod.repo | sudo tee /etc/yum.repos.d/microsoft.repo
```

## Install PowerShell
```console
sudo yum install -y powershell
```

## Start PowerShell
```console
pwsh
```

# Authentication on DC
## URL: http://acrelinux.org/centos-7-autenticando-em-dominio-active-directorysamba4-pelo-realmd-com-winbind/
```console
yum install -y realmd samba-common oddjob-mkhomedir oddjob samba-winbind-clients samba-winbind
```

```console
realm join --client-software=winbind -U juliano.morais.adm pratika.br
```

```console
vim /etc/samba/smb.conf
```
template homedir = /home/%U
winbind use default domain = yes
```console
systemctl restart winbind
systemctl enable winbind
```

## SSH
```console
vim /etc/ssh/sshd_config
```
AllowGroups linuxadmins

## SUDO
#### visudo
```console
%linuxadmins ALL=(ALL) ALL
```

## CentOS
### ifconfig
```console
yumm -y install net-tools
```

### Disable IPv6
## URL: https://www.thegeekdiary.com/centos-rhel-7-how-to-disable-ipv6/
```console
# cat /etc/default/grub
```
GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="ipv6.disable=1 crashkernel=auto rhgb quiet"
GRUB_DISABLE_RECOVERY="true"

```console
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## Verificar documentacao Combo de Servidores no file server
