#### Automação de atualizações do sistema Raspberry Pi OS no Raspberry Pi Zero 2 W e envio de notificação por email com autenticação SASL.

#### Pacotes necessários

>[!IMPORTANT]
> Durante a instalação do Postfix será aberta uma janela onde deverá ser escolhida a opção ```Internet Site``` e depois informado o "mail name" que é o nome do host.
>
> Para confirmar o nome do host pode ser utilizado o comando ```hostname -f```.
>
> Caso a janela de configuração não apareça utilize o comando ```sudo dpkg-reconfigure postfix```, após o término da instalação do pacote.

```
  sudo apt install unattended-upgrades
  sudo apt install postfix
  sudo apt install libsasl2-modules
```
#### Habilitar o Unattended-Upgrades
```
  sudo dpkg-reconfigure --priority=low unattended-upgrades
```
Responder "Yes" na janela que será aberta

>[!NOTE]
> --priority=low foi utilizado para que o dpkg pergunte apenas o necessário para configurar o pacote unattended-upgrades.
>
> Para mais informações sobre o uso de Priorities consulte o DEBCONF (7) ```man 7 debconf ```
>
> Caso não possua o DEBCONF(7), pode instalar com o comando ```sudo apt install debconf-doc```

#### Configurar o Unattended-Upgrades
```
  sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```
Editar o conteúdo do arquivo conforme abaixo:

#### Adicionar texto na seção Unattended-Upgrade::Origins-Pattern{:
```
 "origin=Raspbian,codename=${distro_codename},label=Raspbian";
 "origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";
```

 #### Descomentar e Configurar

 Unattended-Upgrade::AutoFixInterruptedDpkg "true";</br>
 Unattended-Upgrade::InstallOnShutdown "false";</br>
 Unattended-Upgrade::Mail "seuemail@seuprovedordeemail.com";</br>
 Unattended-Upgrade::MailReport "on-change"; (always, only-on-error,on-change)</br>
 Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";</br>
 Unattended-Upgrade::Remove-Unused-Dependencies "true";</br>
 Unattended-Upgrade::Automatic-Reboot "true";</br>
 Unattended-Upgrade::Automatic-Reboot-Time "02:00";</br>
 Unattended-Upgrade::Verbose "false";</br>

### Configurar o envio de email

#### Configurar o arquivo /etc/postfix/sasl/sasl_passwd
>  [!TIP]
>  Se estiver utilizando o Gmail deverá criar uma Senha de App. Para essa opção ser habilitada é necessário ativar a autenticação em duas etapas (2 fatores).
>
>  Caso a pasta sasl em /etc/postfix/ não exista pode criá-la com o comando ```sudo mkdir /etc/postfix/sasl/```

Criar arquivo sasl_passwd
```
  sudo touch /etc/postfix/sasl/sasl_passwd
```

Abrir arquivo sasl_passwd:
```
  sudo nano /etc/postfix/sasl/sasl_passwd
```
Conteúdo do arquivo:
>[!NOTE]
> Ajustar ```[servidor smtp]:587``` conforme provedor de email que estiver utilizando.

```
[servidor smtp]:587 seuemail:suasenha
```
#### Criar banco de dados passwd SASL
> [!IMPORTANT]
> Executar o comando abaixo sempre que alterar o arquivo sasl_passwd
```
  sudo postmap /etc/postfix/sasl/sasl_passwd
```
#### Definir permissões de arquivos
```
  sudo chown root:root /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db
  sudo chmod 0600 /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db
```
#### Configurar o arquivo main.cf

>[!WARNING]
> Não copie o arquivo ```main.cf``` disponível no repositório pois o conteúdo será diferente para cada instalação do postfix.
>
> Em vez disso edite o arquivo original (gerado na instalação do Postfix) de acordo com este tópico.

```
  sudo nano /etc/postfix/main.cf
```
##### Comentar a linha 

  #smtp_tls_security_level=may
  
##### Configurar o relay host

  relayhost = [servidor smtp]:587
  
 ##### Habilitar Autenticação SASL

   Inserir linhas no final do arquivo
  ```
  smtp_sasl_auth_enable = yes
  smtp_sasl_security_options = noanonymous
  smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
  smtp_tls_security_level = encrypt
  smtp_tls_mandatory_ciphers = high
  smtp_tls_mandatory_protocols = >=TLSv1.2, <=TLSv1.3
  smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```
  Reiniciar o Postfix  
```
  sudo systemctl restart postfix.service
```

#### Testar o envio de email
```
  sudo unattended-upgrades -d
```
>[!TIP]
> Se escolheu a opção "on-change" no parâmetro ```Unattended-Upgrade::MailReport "on-change";``` apenas receberá o email se houver alguma "mudança" como atualização, remoção de pacotes, etc...
>
> Para realizar o teste de forma mais assertiva mude o parâmetro para ```Unattended-Upgrade::MailReport "always";``` assim receberá o email informando a execução do unattended-upgrade, mesmo que não haja atualização a ser feita.

### Gestão das atualizações

#### Configurar o intervalo das atualizações

```
  sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

Inserir este conteúdo no arquivo:

>[!NOTE]
> O número entre "aspas" indica o intervalo de dias em que as tarefas do APT serão realizadas. Esta configuração trabalha junto com a configuração de horário do APT (próximo tópico).

```
APT::Periodic::Update-Package-Lists "1"; // Atualiza a lista de pacotes
APT::Periodic::Download-Upgradeable-Packages "1"; // Realiza o Download dos pacotes atualizáveis
APT::Periodic::AutocleanInterval "30"; // Limpa o cache APT
APT::Periodic::Unattended-Upgrade "1"; // Instala os pacotes
```

#### Configurar o horário das atualizações

>[!TIP]
> Não utilize o Cron para agendar as atualizações. O Unattended-Upgrades utiliza a configuração de horário do APT!

Caminho dos arquivos de horário de atualização APT:

**Horário de download**

/lib/systemd/system/apt-daily.timer

**Horário de upgrade**

/lib/systemd/system/apt-daily-upgrade.timer

**Definir o horário de Download**
```
  sudo systemctl edit apt-daily.timer
```
>[!NOTE]
> O comando acima criará o arquivo ```/etc/systemd/system/apt-daily.timer.d/override.conf``` que deverá conter o conteúdo abaixo.

Acrescentar o código
```
  [Timer]
  OnCalendar=
  OnCalendar=08:00 (colocar o horário)
  RandomizedDelaySec=0
```
>[!IMPORTANT]
> O código OnCalendar= é necessário para realizar o override do horário original. Se não colocar este código a configuração do horário não irá funcionar.

Reiniciar o serviço apt-daily.timer
```
  sudo systemctl restart apt-daily.timer
```
Verificar o agendamento
```
  sudo systemctl status apt-daily.timer
```
**Definir o horário de Upgrade**
```
  sudo systemctl edit apt-daily-upgrade.timer
```
>[!NOTE]
> O comando acima criará o arquivo ```/etc/systemd/system/apt-daily-upgrade.timer.d/override.conf``` que deverá conter o conteúdo abaixo.

Acrescentar o código
```
  [Timer]
  OnCalendar=
  OnCalendar=08:20 (colocar o horário)
  RandomizedDelaySec=0
```
>[!IMPORTANT]
> O código OnCalendar= é necessário para realizar o override do horário original. Se não colocar este código a configuração do horário não irá funcionar.

Reiniciar o serviço apt-daily-upgrade.timer
```
  sudo systemctl restart apt-daily-upgrade.timer
```
Verificar o agendamento
```
  sudo systemctl status apt-daily-upgrade.timer
```

>[!WARNING]
> Os procedimentos descritos neste documento foram realizados levando em consideração um contexto operacional específico e com melhores esforços para garantir a Confidencialidade, Integridade, Disponibilidade e Privacidade dos dados. Você é responsável por verificar a adequação de qualquer instrução contida neste documento a seu contexto operacional e requisitos CIA.
  
