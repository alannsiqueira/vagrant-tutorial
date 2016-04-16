#Criando e compartilhando seu ambiente de desenvolvimento com VAGRANT

**Sinta-se encorajado a questionar e contribuir com esse tutorial**

#### Se você estiver usando o ```Windows``` instale algum software que dê suporte a comando UNIX, sugiro o [cmder] ou [gitbash]

>O conteúdo desse tutorial é o resultado de vários tutoriais, há trechos de minha autoria, porem grande parte são recortes de outros tutoriais.
Meu propósito aqui não e ganhar créditos e sim repassar conhecimento.

*Referencias estão citadas no rodapé do mesmo.*

#### Nesse tutorial estarei utilizando para fazer o box,  a versão 14.04 do ubuntu na sua arquitetura de 32 bits, pelos seguintes motivos. 
  - Meu computador não consegue virtualizar arquiteturas de 64 bits.
  - Não obtive sucesso ao tentar no ubuntu 15.10 na arquitetura x32
  - O box ficará mais genérico tendo em vista que várias pessoas (assim como eu), não tem possibilidade de virtualizar tais arquiteturas

#### Versão do virtualbox 4.3.36
- Ao usar o virtualbox 5 não consegui inicializar a rede normalmente, não querendo perder tempo com isso não fui atrás para resolver o mesmo, porem se desejar usar e encontrar a solução, nao deixe de fazer um pull-request

Usuários usando windows 10 relataram erro ao fazer o download de boxes, na busca rápida sobre o assunto
notei que de fato é um bug, sugiro a versão 1.7.4

https://releases.hashicorp.com/vagrant/1.7.4/vagrant_1.7.4.msi

#### Versão Vagrant 1.7.4



## Tabela de Conteúdo

1. [Vagrant, o que é?](#1---vagrant-o-que-é)
1. [Configurando VirtualBox](#2---configurando-virtualbox)
1. [Preparando seu ambiente base](#3---preparando-seu-ambiente-base)
1. [Disponibilizando o box gerado](#4---disponibilizando-o-box-gerado)
1. [Provisionando o box](#5---provisionando-seu-box)
1. [Vagrant Push](#6---vagrant-push)
1. [Vagrant Shared](#7---vagrant-shared)
1. [Dicas](#8---dicas)
1. [Problemas frequentes](#9---problemas-frequentes)
1. [Referencias](#referencias)

## Instalação

Para um setup básico, você precisará instalar o Vagrant e o VirtualBox em sua máquina. A melhor maneira de fazer isso é buscar os pacotes mais atualizados diretamente na [página do Vagrant] e na [página do VirtualBox]. Ambos estão disponíveis para os principais sistemas operacionais. 

## 1 - Vagrant, o que é?

>Vagrant é uma ferramenta para construir e compartilhar ambientes de desenvolvimento. Com um fluxo de trabalho simples e com foco na automação, o Vagrant reduz o tempo de configuração de seu ambiente de desenvolvimento.

O Vagrant é um gerenciador de VMs (máquinas virtuais). Através dele é possível definir o ambiente de desenvolvimento onde seu projeto irá rodar. Com suporte para Mac OSX, Linux e Windows, consegue atender boa parte dos desenvolvedores. Ele utiliza providers, boxes e se necessário provisioners.

#### 1.1.1 - Providers (Provedores)

O Vagrant precisa ter um provider instalado. Ele será onde a máquina virtual contendo seu ambiente irá rodar. O Vagrant dá suporte a providers como VirtualBox, VMware e AWS entre outros.

#### 1.1.2 - Boxes

Afim de evitar o processo tedioso de criar uma VM do zero, o Vagrant trabalha com boxes. Elas são imagens base para ele clonar uma VM. Definir uma box para seu ambiente é o primeiro passo a ser feito na configuração do seu ```Vagrantfile```. Caso a box selecionada já tenha tudo o que você precisa, não é necessário se preocupar com provisioners.

#### 1.1.3 - Provisioners (Provisionadores)

Com provisioners é possível pré-instalar aplicações, definir configurações e realizar toda a parte de ajuste fino de uma box para atender as suas necessidades. É possível utilizar desde shell scripts básicos até sistemas de gerenciamento de configurações como o [Chef] , [Puppet] , [PuPHPet] e [Docker]


### Nota: Executando os Provisionadores

Por padrão, os provisionadores só são executados na primeira vez que você roda um vagrant up, quando o ambiente é criado. Nas próximas vezes que você for usar o ambiente, ele já estará pronto, por isso o Vagrant não irá executar por padrão os provisionadores novamente - isso economiza bastante tempo no dia a dia. Mas há ocasiões em que precisamos executar os provisionadores novamente - principalmente quando estamos criando e ajustando os scripts de provisionamento.

Para executar novamente os provisionadores numa VM que já foi criada, mas está desligada, use vagrant up --provision. Se a máquina já está ligada, você pode usar vagrant provision para executar os provisionadores diretamente, ou vagrant reload --provision para reiniciar a máquina e então executar os provisionadores.

#### 1.1.4 - Usando Ferramentas de Automação

Apesar de parecer a opção mais simples e fácil, Shell script não é a melhor forma de provisionar o seu ambiente de desenvolvimento. Existem ferramentas que foram criadas especificamente para a finalidade de automatizar tarefas em servidores, e elas possuem muitos recursos que você teria de implementar por conta própria caso estivesse preparando seu provisionamento com Shell script - como por exemplo a utilização de templates, comportamento idempotente, modularização, dentre outros.

O Vagrant suporta várias ferramentas de automação. As mais populares são: Puppet, Chef e Ansible, nesta ordem. Não existe uma ferramenta melhor que a outra, mas cada uma tem vantagens e características específicas que podem se adequar melhor para o que você pretende fazer; de uma maneira geral, tudo que pode ser feito em uma ferramenta poderá também ser implementado em outra, mas o nível de complexidade pode variar bastante entre elas.

#### 1.1.5 - Comparação entre ferramentas

Segue um resumo das principais diferenças entre as ferramentas de automações mais usadas como provisionadores do Vagrant, de maneira que você possa compará-las entre si e escolher aquela com a qual você se identifica mais. 

#### Puppet
Syntaxe usada          - linguagem customizada baseada no Ruby                                                                                      
Ordem de execução      - não sequencial - você precisa definir dependências entre as tarefas                                                        
Popularidade          - a mais popular, exceto para desenvolvedores Ruby.                                                                        
Documentação           - um pouco confusa                                                                                                              
Curva de aprendizado   - média                                                                                                                      
Dependências (Vagrant) - não é preciso instalar nenhum pacote adicional para usar Puppet como provisionador                                         


#### Chef
Syntaxe usada            - Ruby                                                                                                                        
Ordem de execução        - sequencial                                                                                                                  
Popularidade             - segunda mais popular, primeira entre desenvolvedores Ruby.                                                                  
Documentação             - complexa!                                                                                                                   
Curva de aprendizado     - alta, você precisa aprender um pouco de Ruby para escrever os scripts                                                      
Dependências (Vagrant)   - não é preciso instalar nenhum pacote adicional para usar Chef como provisionador                                            

#### Ansible
Syntaxe usada         - YAML                                                                                                                          
Ordem de execução     - sequencial                                                                                                                    
Popularidade          - terceira mais popular                                                                                                          
Documentação          - clara e objetiva                                                                                                               
Curva de aprendizado  - pequena                                                                                                                         



*Dependências (Vagrant) - você precisará instalar o Ansible na máquina Host para usá-lo como provisionador* 

#### 1.1.6 - Recursos para Iniciantes
 
Para ter um ponto de partida prático e fácil, você também pode usar uma ferramenta web que gera provisionamentos de ambientes. Temos duas ferramentas atualmente com essa finalidade:

[PuPHPet] - usa [Puppet] como provisionador. Suporta múltiplos provedores e possui uma grande quantidade de recursos, incluindo bancos de dados.

[Phansible] - inspirado pelo [PuPHPet], usa [Ansible] como provisionador, possui recursos básicos, mas já é possível criar um provisionamento de diferentes web servers com PHP, incluindo HHVM.

[VaproBash] - Utiliza o ```shell``` como provisionador, apenas descomentando algumas linhas de código você tem seu box completo

**Como não sou expert nos provisionadores mais complexos, optei por user o [PuPHPet] e [VaproBash] como meus provisionadores, recomendo ambos.** 


#### 1.1.7 - Legal… Mas como funciona?

Quando executamos o comando para o Vagrant subir uma VM, ele lê o arquivo ```Vagrantfile```. Nele estão todas as configurações e definições da VM em questão. O Vagrant inicia uma box no provider, definida no arquivo de configuração. Caso existam mais instruções expressas através de provisioners, ele as executa antes de deixar a máquina disponível.

## 2 - Configurando VirtualBox

*Todos os recursos de hardware previamente definidos poderão ser alterados no VagrantFile, conforme necessidade do desenvolvedor* 

- Criar máquina virtual
  - setar tamanho do hd (alocado dinamicamente)
  - setar quantidade de memoria 
  - desativar recursos que não serão utilizados
  - Instale a distro normalmente.

## 3 - Preparando seu ambiente base

#### 3.1 - Atualizando sua maquina

```bash
sudo apt-get update -y && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y
```

#### 3.2 - Instalando Guest Additions

```bash
sudo apt-get install linux-headers-generic build-essential dkms
sudo /dev/cdrom /media/cdrom
sudo sh /media/cdrom/VBoxLinuxAdditions.run
```

#### 3.3 - Instale os pacotes básicos
 *Ou outros que desejar* 
```bash
sudo apt-get -y install vim git-core
```

#### 3.4 - Setando editor padrão
```bash
sudo update-alternatives --config editor
```

#### 3.5 - Configurando o usuário```vagrant```

*deixa-lo sem senha e os campos em brancos* 

```bash
sudo adduser vagrant  
sudo adduser vagrant sudo
sudo hostname vagrant
sudo groupadd admin
sudo usermod -G admin vagrant
#Delete a senha caso exista
sudo passwd vagrant -d
```

#### 3.6 - Editando o arquivo sudoers
```bash
sudo visudo
```

*Adicione essas linhas conforme exemplo abaixo
```
%admin   ALL=NOPASSWD: ALL
vagrant  ALL=(ALL) NOPASSWD: ALL

```

*Após isso seu arquivo deverá estar parecido com isso:* 
```bash
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=NOPASSWD:ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
vagrant ALL=(ALL) NOPASSWD:ALL

```

#### 3.7 - Instalando OpenSSH e Vagrant keys

```bash
sudo apt-get install -y openssh-server

mkdir ~/.ssh/
cd ~/.ssh
wget http://github.com/mitchellh/vagrant/raw/master/keys/vagrant
wget http://github.com/mitchellh/vagrant/raw/master/keys/vagrant.pub
mv vagrant.pub authorized_keys
sudo chmod 0700 /home/vagrant/.ssh
sudo chmod 0600 /home/vagrant/.ssh/authorized_keys
sudo chown -R vagrant:vagrant /home/vagrant/.ssh
```


#### 3.8 - Verificando se as configurações SSH estã corretas

``` bash
 sudo vim /etc/ssh/sshd_config
 ```

*Verifique as seguintes linhas no arquivo aberto:* 
``` 
- Port 22
- PubKeyAuthentication yes
- AuthorizedKeysFile %h/.ssh/authorized_keys
- PermitEmptyPasswords no
```

**Salve e feche o arquivo**

####3.9 - Compactando espaços

```bash
- sudo dd if=/dev/zero of=/EMPTY bs=1M
- sudo rm -f /EMPTY
```


#### 3.10 - Limpando historico, cache e desligando

```bash
sudo apt-get clean && history -c && sudo shutdown -h now
```


## 4 - Disponibilizando o box gerado

```bash
sudo shutdown -h now 
vagrant package --base nome-da-sua-vm --output /caminho-da-sua-pasta/nome-do-seu-box.box
```

#### 4.1 - Importando o box localmente

*Notem que são 2 contra-barras no file:\\ e 2 barras no //caminho//para//seu//arquivo.box* 

``` vagrant box add  nome-do-seu-box.nome.da.distro.versao-plataforma ``` **file:\\**//caminho-para-seu-box.box


#### 4.2 - Importando no Vagrant Cloud

O repositório de onde o Vagrant baixa as VMs (boxes) é o [Vagrant Cloud].
Dessa forma preciso [criar uma conta nesse serviço] para poder utilizá-lo e após criá-la, posso configurar meus próprios boxes.

O site Vagrant Cloud apresenta um passo a passo muito simples para que possamos publicar nossos próprios boxes.É só segui-lo, não tem segredos e os passos são descritos na página [Creating a new Box]
Num dos últimos passos é necessário informar uma URL para a localização do box (o arquivo com a extensão .box).Por fim, depois de ter meu box publicado no Vagrant, eu posso utilizá-lo através do comando a seguir:

vagrant init alannsiqueira/webschool-box-ubuntu-14.04-x32

Todas as VMs que criamos no Vagrant Cloud também ficam acessíveis através de uma URL contendo nosso nome de usuário no Vagrant Cloud.
A minha URL é https://vagrantcloud.com/alannsiqueira/.


## 5 - Provisionando seu box

#### 5.1 - Comandos básicos:

Comando   | Descrição  | Uso
----------------------------|-------------------------------------------------------| ----------------------------------------------------
vagrant up                  | Inicializa a maquia virtual e executa o provisioner.  |Quando vamos começar a subir nosso ambiente.
vagrant reload              | Reinicia a máquina virtual.                           |Necessário caso haja alteração no vagrantfile.
vagrant provision           | Executa o provisioner.                                |Quando o script de provisionamento for alterado.
vagrant init                | Gera um novo vagrantfile baseado em uma box.          |Quando estamos iniciando nosso projeto.
vagrant halt                | Desliga a máquina virtual.                            |Quando vamos desligar a maquina virtual.
vagrant destroy             | Destrói a máquina virtual                             |Quando vamos limpar tudo e começar de novo
vagrant suspend             | Pausa a máquina virtual                               |Quando queremos parar e manter o estado atual da maquina
vagrant resume              | Retira a pausa da máquina virtual                     |Quando queremos continuar a partir do estado salvo
vagrant ssh                 | Acessa a máquina virtual via ssh                      |Quando queremos executar comandos manuais

#### 5.2 - O VagrantFile

*Vejamos como está o VagrantFile inicialmente*

```ruby

# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "base"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end


```

## 5.2.1 - Estudando o conteudo do arquivo

### config.vm

*As configurações dentro do config.vm modificam toda a parte de alocação de hardware e virtualização da VM.* 

####config.vm.box
Onde é configurado o box. Box é uma imagem que o Vagrant utiliza para clonar e criar a Virtual Machine,
ao invés de termos que baixar manualmente. Temos inúmeras distribuições de sistemas para usar como nosso box,
você pode buscar qual você deseja aqui;
####config.vm.network
Onde temos a configuração de rede para acessar o nosso aplicativo pela virtual machine;
####config.vm.provision
Onde temos a configuração para o provisionamento que será feito;
####config.vm.provider
Onde temos a configuração para o provider que será utilizado, default é o Virtual Box;
####config.vm.synced_folder
Onde é passado quais pastas queremos que seja sincronizada com a nossa VM, por default, a pasta onde temos o Vagrantfile já é sincronizada automaticamente sobre a pasta /vagrant/ na VM;
####config.vm.communicator
Onde é configurado como será feita a comunicação entre a VM e a maquina local, por default, é utilizado ssh;
####config.vm.post_up_message
Onde é configurado qual mensagem será apresentada após o comando vagrant up, pode ser muito útil para apresentar informações sobre como acessar todos os componentes do sistema;

### 5.2.2 - config.ssh

*As configurações dentro do config.ssh servem para configurar como o Vagrant irá acessar a VM via ssh.* 

####config.ssh.username
Onde define qual usuário será configurado pra acessar a VM via ssh, por default, o usuário é o ‘vagrant’;
Onde define a senha para o usuário que for usado para conectar via ssh. É recomendável utilizar private_key_path para a configuração de senha;
####config.ssh.sudo_command
Onde define que será por padrão a utilização do sudo nos comandos de shell;
####config.winrm
As configurações dentro do config.winrm servem para configurar como o Vagrant irá acessar a VM com o Windows.

####config.winrm.username
Onde define qual usuário será configurado pra acessar a VM via winrm, por default, o usuário é o ‘vagrant’;
####config.winrm.password
Onde define a senha para o usuário que for usado para conectar via winrm, por default, a senha é ‘vagrant’;


## 5.3 - Provisionando 

Seguindo os conceitos abordados, utilizarei o [VaproBash] para provisionar meu box.

- Baixando o VagrantFile 
```bash
$ curl -L http://bit.ly/vaprobash > Vagrantfile
```

Procurando no arquivo baixado o seguinte termo config.vm.box = "ubuntu/trusty64" substituir para config.vm.box = "nome-da-sua-base-que-foi-importada-anteriormente"

O VagrantFile baixado e totalmente intuitivo, qualquer pessoa com conhecimento minimo em bash/shell conseguirá entender.
Funciona da seguinte maneira, você adiciona os parametros desejados ex: **hostname** , **IP**  , **memoria**, **processador**, **usuario/senha** 
e na area mais perto do final do arquivo, você descomenta as linhas que deseja provisionar (*instalar*)

Após estar tudo configurado, basta somendo inicializar o box com o comando ```vagrant up --provision```

*Vá tomar um café, da uma volta .. contribuir para algum projeto ... o negocio demora um pouco!* 

**Meu arquivo fico assim, um box totalmente preparado para desenvolvimento MEAN** 

```ruby

# -*- mode: ruby -*-
# vi: set ft=ruby :


# Config Github Settings
github_username = "fideloper"
github_repo     = "Vaprobash"
github_branch   = "1.4.2"
github_url      = "https://raw.githubusercontent.com/#{github_username}/#{github_repo}/#{github_branch}"

# Because this:https://developer.github.com/changes/2014-12-08-removing-authorizations-token/
# https://github.com/settings/tokens
github_pat          = ""

# Server Configuration

hostname        = "webschool-box"

# Set a local private network IP address.
# See http://en.wikipedia.org/wiki/Private_network for explanation
# You can use the following IP ranges:
#   10.0.0.1    - 10.255.255.254
#   172.16.0.1  - 172.31.255.254
#   192.168.0.1 - 192.168.255.254
server_ip             = "192.168.56.100"
server_cpus           = "1"   # Cores
server_memory         = "384" # MB
server_swap           = "768" # Options: false | int (MB) - Guideline: Between one or two times the server_memory

# UTC        for Universal Coordinated Time
# EST        for Eastern Standard Time
# CET        for Central European Time
# US/Central for American Central
# US/Eastern for American Eastern
server_timezone  = "UTC"

# Database Configuration
mysql_root_password   = "root"   # We'll assume user "root"
mysql_version         = "5.5"    # Options: 5.5 | 5.6
mysql_enable_remote   = "false"  # remote access enabled when true
pgsql_root_password   = "root"   # We'll assume user "root"
mongo_version         = "2.6"    # Options: 2.6 | 3.0
mongo_enable_remote   = "false"  # remote access enabled when true

# Languages and Packages
php_timezone          = "UTC"    # http://php.net/manual/en/timezones.php
php_version           = "5.6"    # Options: 5.5 | 5.6
ruby_version          = "latest" # Choose what ruby version should be installed (will also be the default version)
ruby_gems             = [        # List any Ruby Gems that you want to install
  #"jekyll",
  #"sass",
  #"compass",
]

go_version            = "latest" # Example: go1.4 (latest equals the latest stable version)

# To install HHVM instead of PHP, set this to "true"
hhvm                  = "false"

# PHP Options
composer_packages     = [        # List any global Composer packages that you want to install
  #"phpunit/phpunit:4.0.*",
  #"codeception/codeception=*",
  #"phpspec/phpspec:2.0.*@dev",
  #"squizlabs/php_codesniffer:1.5.*",
]

# Default web server document root
# Symfony's public directory is assumed "web"
# Laravel's public directory is assumed "public"
public_folder         = "/vagrant"

laravel_root_folder   = "/vagrant/laravel" # Where to install Laravel. Will `composer install` if a composer.json file exists
laravel_version       = "latest-stable" # If you need a specific version of Laravel, set it here
symfony_root_folder   = "/vagrant/symfony" # Where to install Symfony.

nodejs_version        = "latest"   # By default "latest" will equal the latest stable version
nodejs_packages       = [          # List any global NodeJS packages that you want to install
  "grunt-cli",
  "express"
  "gulp",
  "bower",
  "yo",
]

# RabbitMQ settings
#attr_reader :attr_namesabbitmq_user = "user"
#rabbitmq_password = "password"

#sphinxsearch_version  = "rel22" # rel20, rel21, rel22, beta, daily, stable


Vagrant.configure("2") do |config|

  # Set server to Ubuntu 14.04
  config.vm.box = "webschool-Ubuntu-14.04-x32"

  config.vm.define "webschool-box" do |vapro|
  end

  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = false
  end

  # Create a hostname, don't forget to put it to the `hosts` file
  # This will point to the server's default virtual host
  # TO DO: Make this work with virtualhost along-side xip.io URL
  config.vm.hostname = hostname

  # Create a static IP
  if Vagrant.has_plugin?("vagrant-auto_network")
    config.vm.network :private_network, :ip => "0.0.0.0", :auto_network => true
  else
    config.vm.network :private_network, ip: server_ip
    config.vm.network :forwarded_port, guest: 80, host: 8080
    config.vm.network :forwarded_port, guest: 22, host: 7222

  end

  # Enable agent forwarding over SSH connections
  config.ssh.forward_agent = true

  # Use NFS for the shared folder
  #config.vm.synced_folder ".", "/vagrant",
  #  id: "core",
  #  :nfs => true,
  #  :mount_options => ['nolock,vers=3,udp,noatime,actimeo=2,fsc']

  # Replicate local .gitconfig file if it exists
  if File.file?(File.expand_path("~/.gitconfig"))
    config.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"
  end

  # If using VirtualBox
  config.vm.provider :virtualbox do |vb|

    vb.name = hostname

    # Set server cpus
    vb.customize ["modifyvm", :id, "--cpus", server_cpus]

    # Set server memory
    vb.customize ["modifyvm", :id, "--memory", server_memory]

    # Set the timesync threshold to 10 seconds, instead of the default 20 minutes.
    # If the clock gets more than 15 minutes out of sync (due to your laptop going
    # to sleep for instance, then some 3rd party services will reject requests.
    vb.customize ["guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 10000]

    # Prevent VMs running on Ubuntu to lose internet connection
    # vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    # vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]

  end

  # If using VMWare Fusion
  config.vm.provider "vmware_fusion" do |vb, override|
    override.vm.box_url = "http://files.vagrantup.com/precise64_vmware.box"

    # Set server memory
    vb.vmx["memsize"] = server_memory

  end

  # If using Vagrant-Cachier
  # http://fgrehm.viewdocs.io/vagrant-cachier
  if Vagrant.has_plugin?("vagrant-cachier")
    # Configure cached packages to be shared between instances of the same base box.
    # Usage docs: http://fgrehm.viewdocs.io/vagrant-cachier/usage
    config.cache.scope = :box

    config.cache.synced_folder_opts = {
        type: :nfs,
        mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    }
  end

  # Adding vagrant-digitalocean provider - https://github.com/smdahlen/vagrant-digitalocean
  # Needs to ensure that the vagrant plugin is installed
  config.vm.provider :digital_ocean do |provider, override|
    override.ssh.private_key_path = '~/.ssh/id_rsa'
    override.ssh.username = 'vagrant'
    override.vm.box = 'digital_ocean'
    override.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"

    provider.token = 'YOUR TOKEN'
    provider.image = 'ubuntu-14-04-x64'
    provider.region = 'nyc2'
    provider.size = '512mb'
  end

  ####
  # Base Items
  ##########

  # Provision Base Packages
  config.vm.provision "shell", path: "#{github_url}/scripts/base.sh", args: [github_url, server_swap, server_timezone]

  # optimize base box
  config.vm.provision "shell", path: "#{github_url}/scripts/base_box_optimizations.sh", privileged: true

  # Provision PHP
  config.vm.provision "shell", path: "#{github_url}/scripts/php.sh", args: [php_timezone, hhvm, php_version]

  # Enable MSSQL for PHP
  # config.vm.provision "shell", path: "#{github_url}/scripts/mssql.sh"

  # Provision Vim
   config.vm.provision "shell", path: "#{github_url}/scripts/vim.sh", args: github_url

  # Provision Docker
  # config.vm.provision "shell", path: "#{github_url}/scripts/docker.sh", args: "permissions"

  ####
  # Web Servers
  ##########

  # Provision Apache Base
  # config.vm.provision "shell", path: "#{github_url}/scripts/apache.sh", args: [server_ip, public_folder, hostname, github_url]

  # Provision Nginx Base
  # config.vm.provision "shell", path: "#{github_url}/scripts/nginx.sh", args: [server_ip, public_folder, hostname, github_url]


  ####
  # Databases
  ##########

  # Provision MySQL
  #config.vm.provision "shell", path: "#{github_url}/scripts/mysql.sh", args: [mysql_root_password, mysql_version, mysql_enable_remote]

  # Provision PostgreSQL
  #config.vm.provision "shell", path: "#{github_url}/scripts/pgsql.sh", args: pgsql_root_password

  # Provision SQLite
  #config.vm.provision "shell", path: "#{github_url}/scripts/sqlite.sh"

  # Provision RethinkDB
  # config.vm.provision "shell", path: "#{github_url}/scripts/rethinkdb.sh", args: pgsql_root_password

  # Provision Couchbase
  # config.vm.provision "shell", path: "#{github_url}/scripts/couchbase.sh"

  # Provision CouchDB
  # config.vm.provision "shell", path: "#{github_url}/scripts/couchdb.sh"

  # Provision MongoDB
   config.vm.provision "shell", path: "#{github_url}/scripts/mongodb.sh", args: [mongo_enable_remote, mongo_version]

  # Provision MariaDB
  # config.vm.provision "shell", path: "#{github_url}/scripts/mariadb.sh", args: [mysql_root_password, mysql_enable_remote]

  # Provision Neo4J
   config.vm.provision "shell", path: "#{github_url}/scripts/neo4j.sh"

  ####
  # Search Servers
  ##########

  # Install Elasticsearch
  # config.vm.provision "shell", path: "#{github_url}/scripts/elasticsearch.sh"

  # Install SphinxSearch
  # config.vm.provision "shell", path: "#{github_url}/scripts/sphinxsearch.sh", args: [sphinxsearch_version]

  ####
  # Search Server Administration (web-based)
  ##########

  # Install ElasticHQ
  # Admin for: Elasticsearch
  # Works on: Apache2, Nginx
  # config.vm.provision "shell", path: "#{github_url}/scripts/elastichq.sh"


  ####
  # In-Memory Stores
  ##########

  # Install Memcached
  # config.vm.provision "shell", path: "#{github_url}/scripts/memcached.sh"

  # Provision Redis (without journaling and persistence)
  # config.vm.provision "shell", path: "#{github_url}/scripts/redis.sh"

  # Provision Redis (with journaling and persistence)
  # config.vm.provision "shell", path: "#{github_url}/scripts/redis.sh", args: "persistent"
  # NOTE: It is safe to run this to add persistence even if originally provisioned without persistence


  ####
  # Utility (queue)
  ##########

  # Install Beanstalkd
  # config.vm.provision "shell", path: "#{github_url}/scripts/beanstalkd.sh"

  # Install Heroku Toolbelt
   config.vm.provision "shell", path: "https://toolbelt.heroku.com/install-ubuntu.sh"

  # Install Supervisord
  # config.vm.provision "shell", path: "#{github_url}/scripts/supervisord.sh"

  # Install Kibana
  # config.vm.provision "shell", path: "#{github_url}/scripts/kibana.sh"

  # Install ØMQ
  # config.vm.provision "shell", path: "#{github_url}/scripts/zeromq.sh"

  # Install RabbitMQ
  # config.vm.provision "shell", path: "#{github_url}/scripts/rabbitmq.sh", args: [rabbitmq_user, rabbitmq_password]

  ####
  # Additional Languages
  ##########

  # Install Nodejs
   config.vm.provision "shell", path: "#{github_url}/scripts/nodejs.sh", privileged: false, args: nodejs_packages.unshift(nodejs_version, github_url)

  # Install Ruby Version Manager (RVM)
   config.vm.provision "shell", path: "#{github_url}/scripts/rvm.sh", privileged: false, args: ruby_gems.unshift(ruby_version)

  # Install Go Version Manager (GVM)
  # config.vm.provision "shell", path: "#{github_url}/scripts/go.sh", privileged: false, args: [go_version]

  ####
  # Frameworks and Tooling
  ##########

  # Provision Composer
  # You may pass a github auth token as the first argument
  # config.vm.provision "shell", path: "#{github_url}/scripts/composer.sh", privileged: false, args: [github_pat, composer_packages.join(" ")]

  # Provision Laravel
  #config.vm.provision "shell", path: "#{github_url}/scripts/laravel.sh", privileged: false, args: [server_ip, laravel_root_folder, public_folder, laravel_version]

  # Provision Symfony
  #config.vm.provision "shell", path: "#{github_url}/scripts/symfony.sh", privileged: false, args: [server_ip, symfony_root_folder, public_folder]

  # Install Screen
   config.vm.provision "shell", path: "#{github_url}/scripts/screen.sh"

  # Install Mailcatcher
  # config.vm.provision "shell", path: "#{github_url}/scripts/mailcatcher.sh"

  # Install git-ftp
   config.vm.provision "shell", path: "#{github_url}/scripts/git-ftp.sh", privileged: false

  # Install Ansible
  # config.vm.provision "shell", path: "#{github_url}/scripts/ansible.sh"

  # Install Android
  # config.vm.provision "shell", path: "#{github_url}/scripts/android.sh"

  ####
  # Local Scripts
  # Any local scripts you may want to run post-provisioning.
  # Add these to the same directory as the Vagrantfile.
  ##########
  # config.vm.provision "shell", path: "./local-script.sh"

end


```

  
## 6 - Vagrant push

- Criar conteudo

## 7 - Vagrant shared

- adicionar conteudo

## 8 - Dicas

*Usar um provisionador para montar seu ambiente facilitará muito sua vida para montagem e manutenção do seu box* 

##### Configure um ambiente de desenvolvimento IONIC

http://matheuslao.github.io/2015/06/09/ambiente-desenvolvimento-ionic-framework-ubuntu-vagrant/

##### Crie um ```alias``` para poder acessar sua maquina localmente mais rapidamente de qualquer lugar

```bash
alias vm-ssh='ssh vagrant@nome-da-sua-vm -i ~/.vagrant.d/insecure_private_key
```

##### Crie um ```script``` desligar e iniciar sua VM quando for desligar/suspender/iniciar sua maquina HOST

http://blog.icaromh.com/vagrant-halt-antes-de-desligar-o-sistema/

## 9 - Problemas frequentes

### SSH está muito lento!
    - R: Verifique se não esta acessando através de um DNS,
     tente acessar localmente, através de sua private key

### Não consigo baixar boxes alternativos
    - R: Verifique se o curl esta devidamente instalado e setado na variável de ambiente

### Ja configurei meu box, porem não tenho acesso a internet (Windows)

##### Sei que não e a melhor forma, porem, foi rápida e eficaz para mim.

- Adicionar essa linha ```sudo ifdown eth0 && sudo ifup eth0``` no arquivo em  ``` /etc/rc.local``` 

- Editar arquivo para retirar delay da configuração de rede na VM

    ``` bash
     sudo vim /etc/init/failsafe.conf
    ```
   
    - Trocar a linha    
    ```sleep 5```

    - E comentar as linhas
    ```$PLYMOUTH message --text="Waiting for network configuration..." || :```
    ```sleep 40```         
    ```$PLYMOUTH message --text="Waiting up to 60 more seconds for network configuration..." || :```
    ```sleep 59```

###### Seu arquivo deverá ficar parecer igual a esse


```bash
# failsafe

description "Failsafe Boot Delay"
author "Clint Byrum <clint@ubuntu.com>"

start on filesystem and net-device-up IFACE=lo
stop on static-network-up or starting rc-sysinit

emits failsafe-boot

console output

script
        # Determine if plymouth is available
        if [ -x /bin/plymouth ] && /bin/plymouth --ping ; then
                PLYMOUTH=/bin/plymouth
        else
                PLYMOUTH=":"
        fi

    # The point here is to wait for 2 minutes before forcibly booting 
    # the system. Anything that is in an "or" condition with 'started 
    # failsafe' in rc-sysinit deserves consideration for mentioning in
    # these messages. currently only static-network-up counts for that.

        sleep 5

    # Plymouth errors should not stop the script because we *must* reach
    # the end of this script to avoid letting the system spin forever
    # waiting on it to start.
#       $PLYMOUTH message --text="Waiting for network configuration..." || :
#       sleep 40

#       $PLYMOUTH message --text="Waiting up to 60 more seconds for network configuration..." || :
#       sleep 59
#       $PLYMOUTH message --text="Booting system without full network configuration..." || :

    # give user 1 second to see this message since plymouth will go
    # away as soon as failsafe starts.
        sleep 1
    exec initctl emit --no-wait failsafe-boot
end script

post-start exec logger -t 'failsafe' -p daemon.warning "Failsafe of 120 seconds reached."


```

# Referencias

- http://nandovieira.com.br/usando-o-vagrant-como-ambiente-de-desenvolvimento-no-windows
- http://alancastro.org/2014/02/08/criando-box-centos-6-para-vagrant-com-virtualbox.html
- http://aruizca.com/steps-to-create-a-vagrant-base-box-with-ubuntu-14-04-desktop-gui-and-virtualbox/
- http://sysadm.pp.ua/linux/sistemy-virtualizacii/vagrant-box-creation.html
- http://www.vagrantup.com

[Chef]:https://www.chef.io/chef/
[Puppet]:https://puppetlabs.com/
[PuPHPet]:https://puphpet.com/
[Docker]:https://www.docker.com/
[gitbash]: http://git-scm.com/download/win
[cmder]: http://cmder.net
[Vagrant Cloud]: https://atlas.hashicorp.com/boxes/search?utm_source=vagrantcloud.com&vagrantcloud=1
[criar uma conta nesse serviço]:https://vagrantcloud.com/account/new
[Creating a new Box]:https://atlas.hashicorp.com/boxes/new
[página do Vagrant]:https://www.vagrantup.com/downloads.html
[página do VirtualBox]:https://www.virtualbox.org/wiki/Downloads
[VaproBash]:https://fideloper.github.io/Vaprobash/
