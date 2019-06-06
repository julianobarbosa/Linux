
########  INÍCIO  DO MÓDULO DE PROVISIONAMENTO DO DC  ########

AULA 1 - INSTALANDO O SAMBA


AULA 2 - PREPARAÇÃO DO SERVIDOR, ORIENTAÇÕES SOBRE OS PROGRAMAS UTILIZADOS E INSTALAÇÃO DAS BIBLIOTECAS DO SAMBA

Atualize os repositórios do debian
	
	apt-get update
	
	
Para uso do Putty e winscp, é necessário realizar a instalação do ssh no proprio servidor que desejamos acessar.

	apt-get install ssh
	
	
Instalando as dependências do samba
	

	apt-get install ntp acl attr autoconf bind9utils bison build-essential debhelper dnsutils docbook-xml docbook-xsl flex gdb libjansson-dev krb5-user libacl1-dev libaio-dev libarchive-dev libattr1-dev libblkid-dev libbsd-dev libcap-dev libcups2-dev libgnutls28-dev libgpgme11-dev libjson-perl libldap2-dev libncurses5-dev libpam0g-dev libparse-yapp-perl libpopt-dev libreadline-dev nettle-dev perl perl-modules-5.24 libsystemd-dev pkg-config python-all-dev python-crypto python-dbg python-dev python-dnspython python3-dnspython python-gpgme python3-gpgme python-markdown python3-markdown python3-dev xsltproc zlib1g-dev -y

	
Cria a pasta arquivos

	cd /home/administrador/
	mkdir Arquivos
	
Acessa a pasta
	cd Arquivos
	
	
Realizando download do samba

	wget https://ftp.samba.org/pub/samba/stable/samba-4.8.2.tar.gz

	tar zxvf samba-4.8.2.tar.gz



AULA 3 - INSTALANDO O SAMBA COMPILADO


	
Acessando diretório:

	cd /home/administrador/Arquivos/samba-4.8.2
	
	
	 ./configure --prefix=/opt/samba -j 5 
	 make -j 5
	 make install -j 5
	
	
	
Verificando o suporte a ACL


	cat /boot/config-4.9.0-3-amd64 | grep _ACL
	cat /boot/config-4.9.0-3-amd64 | grep FS_SECURITY


Configurando as variáveis de ambiente

	samba-tool

	export PATH=$PATH:'/opt/samba/bin:/opt/samba/sbin'
	echo 'export PATH=$PATH:"/opt/samba/bin:/opt/samba/sbin' >> ~/.bashrc
	echo 'export PATH=$PATH:"/opt/samba/bin:/opt/samba/bin' >> ~/.bashrc

	samba-tool




AULA 4 - CONFIGURANDO SERVIDOR E PROVISIONANDO COMO DC 



ATENÇÃO: Lembre de fazer o snapshot da vm com o samba instalado, conforme mostrado nesta aula

Para verificar o ip do servidor
	
	ip addr ou ifconfig 

Configurando a rede do servidor

	nano /etc/network/interfaces

	address 10.1.139.2
	netmask 255.255.255.0
	gateway 10.1.139.1

Reinicia o servidor para pegar as configurações:
	
	reboot

verifica arquivo se está ok com o comando abaixo:

	cat /etc/network/interfaces


Confere o arquivo /etc/hosts


nano /etc/hosts
	
	127.0.0.1	localhost
	10.1.139.2	SRVDC001.EMPRESA.SMB.COM.BR		SRVDC001
	
		
Configura o resolv.conf
	
	nano /etc/resolv.conf
	
	domain empresa.smb.com.br
	search empresa.smb.com.br
	nameserver 10.1.139.2
	nameserver 10.1.139.1
	
	
Obs.: Para bloquear o arquivo para que não seja alterado ao reiniciar, execute o comando abaixo:

	chattr +i /etc/resolv.conf
	
	
	hostname

	hostname -f


Se tudo estiver ok, efetua o provisionamento

	samba-tool domain provision --realm=empresa.smb.com.br --use-rfc2307 --domain=empresa --dns-backend=SAMBA_INTERNAL --adminpass=SuaSenha --server-role=dc


Fazendo um backup do arquivo original do krb5.conf

	cp /etc/krb5.conf /etc/krb5.conf


	ln -sf /opt/samba/private/krb5.conf /etc/krb5.conf

	cat /etc/krb5.conf

Deve aparecer as informações abaixo:

	[libdefaults]
		default_realm = EMPRESA.SMB.COM.BR
		dns_lookup_realm = false
		dns_lookup_kdc = true


Executando o samba

	/opt/samba/sbin/samba


Verificando a execução do  samba

	ps ax | egrep "samba|smbd|nmbd|winbindd"




##Efetua os testes abaixo:##

	smbclient -L localhost -U%



	smbclient //localhost/netlogon -UAdministrator -c 'ls'


	host -t SRV _ldap._tcp.empresa.smb.com.br.


	host -t SRV _kerberos._udp.empresa.smb.com.br.


	host -t A SRVDC001.empresa.smb.com.br.



	kinit administrator


	klist


Tira um novo snapshot




Aula 5 - CONFIGURANDO O SAMBA PARA SUBIR JUNTO COM O SISTEMA

	OBS.: Copie o arquivo samba-ad-dc.service.tar para o seu windows 8
	
Caso precise de permissõs

	cd /home/administrador/
	
	chmod 777 Arquivos

Acessa a pasta pelo terminal
	
	cd Arquivos 
	
Descompacta

	tar xvzf samba-ad-dc.service.tar


Como encontrar o samba.pid

	find / -iname samba.pid
	
	
Copia para /etc/systemd/system
	
	cp samba-ad-dc.service /etc/systemd/system


Habilitando o samba no boot:

	systemctl enable samba-ad-dc.service
	
	
Verificando os serviços do samba

	ps ax | egrep "samba|smbd|nmbd|winbindd"

Reinicia o servidor
	
	reboot


	ps ax | egrep "samba|smbd|nmbd|winbindd"
	

Tira um novo snapshot

	shutdown now
	
Tira o snapshot





AULA 6 - CONFIGURANDO O WINBIND




Verificando a plataforma ou arquiquetura do pc

uname -a
			
	
Verificando onde estão as bibliotecas do samba.

	/opt/samba/sbin/smbd -b | grep LIBDIR


ATENÇÃO, OBSERVE A ARQUITETURA DO SISTEMA

Criando links entre as bibliotecas do samba com as bibliotecas do sistema

	ln -s /opt/samba/lib/libnss_winbind.so.2 /lib/x86_64-linux-gnu/
	ln -s /lib/x86_64-linux-gnu/libnss_winbind.so.2 /lib/x86_64-linux-gnu/libnss_winbind.so
		


Recarregando as bibliotecas

	ldconfig
	

Editando o arquivo /etc/nsswitch.conf
	
	nano /etc/nsswitch.conf

		
	passwd: files winbind
	group:  files winbind
	shadow: files

		
Recarregando o samba

	smbcontrol all reload-config
		
	
Executando os testes a seguir:
		
getent passwd adminsitrator

	wbinfo -u
	wbinfo -g
		

Tira o Snapshot



AULA 7 - FAZENDO DOWNLOAD DO RSAT, INSTALANDO E HABILITANDO NO WINDOWS 8 E INGRESSANDO WINDOWS 8 NO DOMÍNIO

	https://www.microsoft.com/pt-BR/download/details.aspx?id=39296

No pfsense, na aba Services, adicione as informações do domínio.

Ingresse o windows ao domínio

Para resolver problemas de internet acesse
 
	nano /opt/samba/etc/smb.conf
	
	Em dns forwarder = 10.1.139.2
	
	Troque pelo ip	    10.1.139.1	

salve

Recarregando configurações.

	smbcontrol all reload-config
	
Reiniciando o samba

	systemctl restart samba-ad-dc.service

	
	
	
	
