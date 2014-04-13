---
layout: post
title: "Configurando Wireless no Ubuntu para Notebook HP Pavilion dv2610us"
date: 2014-04-13 14:47:35 -0300
comments: true
categories: [linux, ubuntu, wireless, dv2610us]
---

Após migrar meu notebook (HP dv2610us) para Ubuntu 7.10, a única coisa que me deixava a desejar era o funcionamento do wireless, após vários e vários dias pesquisando e testando diversos drivers e maneiras de reconhecer a mini-PCI Broadcom 94311 que vem junto a ele, consegui finalmente desfrutar do acesso wireless via Linux.

Abaixo segue um breve tutorial de como fazer essa mágica:

**1° Passo:** (Instalar o ndiswrapper)

Ndiswrapper é um emulador de drivers Windows para Linux, seu principal objetivo é fazer com que hardware baseado em Windows possam ser ativado no Linux. Ele também pode ser utilizado para outros hardware além de dispositivos wireless como: WebCam, Modems e Roteadores USB entre outros.

Para instalar o ndiswrapper é só entrar no seu terminal e digitar:

** $ sudo apt-get install ndiswrapper-common ** 

e 

** $ sudo apt-get install ndiswrapper-utils-1.9 ** 

** 2° Passo: ** (Desabilitar o driver nativo pre-instalado)
No Ubuntu já existe um drive Broadcom 43xx previamente instalado que deve ser removido pelo simples motivo de não funcionar para a mini-PCI que vem junto ao dv2610us.

Para desabilitar execute o seguinte comando

** $ echo ‘blacklist bcm43xx’ | sudo tee -a /etc/modprobe.d/blacklist ** 

e 

** $ sudo rmmod bcm43xx **

** 3° Passo: ** (Baixar o drive Broadcom compatível)
Esse é o ponto que me deixa mais indignado, pela lógica e muito provavelmente, você deve estar imaginando “isso se você já não foi logo fazendo :) ” é só ir na site da HP e fazer o download do drive disponível para esse módulo e pronto!!! É ai que você se engana hehehe. Não sei por qual motivo o drive da HP que deveria funcionar não funciona, então, após vários dias de pesquisa, encontrei um site “segue link no final do post” que ensinava a instalar essa mesma mini-PCI com o drive da Dell e então após os meus testes…… Uhuuu, funcionou.

Então para fazer o download

** $ wget http://ftp.us.dell.com/network/R151517.EXE **

Após o download temos que descompactar

** $ unzip -a R151517.EXE **

** 4° Passo: ** (Instalar e ativar o drive no ndiswrapper)
Após descompactar em um diretório temporário, por exemplo “temp”, execute os comandos:

** $ sudo ndiswrapper -i temp/DRIVE/bcmwl5.sys **

E depois temos que iniciar o módulo

**$ sudo ndiswrapper -m** 

**$ sed -e ‘s/RadioState|1/RadioState|0/’ /etc/ndiswrapper/bcmwl5/*.con**

**$ sudo modprobe ndiswrapper**

Reinicie o computador para verificar se realmente o ndiswrapper carregou automática e execute o seguinte comando para listar os pontos de acesso wireless

** $ iwlist scanning **

** 6° e Último Passo: ** (configurando a rede)
Como todo bom usuário Linux, configure a rede na mão com os seguinte comandos:

**$ sudo iwconfig wlan0 essid NOME_DO_ACESSPOINT** “nome listado pelo comando iwlist scanning”

**$ sudo iwconfig wlan0 enc CHAVE** “chave de acesso se utiliza WEP ou WPA – opcional caso venha utilizar rede segura ou pelo menos difícil de quebrar” 
** $ sudo dhclient wlan0 ** “configura para pegar automaticamente um ip fornecido pelo acess point”

~~**$ sudo ifconfig wlan0 down ** “desliga interface wlan0”**~~

~~**$ sudo ifconfig wlan0 up ** “inicia interface wlan0”**~~

**$ sudo iwconfig commit ** “grava alterações”**

E pronto, você já está liberado para utilizar sua rede wireless.

Referência: [http://www.vivaolinux.com.br/dicas/verDica.php?codigo=9358](http://www.vivaolinux.com.br/dicas/verDica.php?codigo=9358)