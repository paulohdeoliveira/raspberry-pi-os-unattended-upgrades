#### Automação de atualizações do sistema Raspbian (debian bookworm) no Raspberry Pi e envio de notificação por email com autenticação SASL.

#### Pacotes necessários
```
  sudo apt install unattended-upgrades
  sudo apt install postfix
  sudo apt install libsasl2-modules
```
#### Configurar o Unattended-Upgrades
```
  sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```
Edite o contéudo do arquivo conforme abaixo:

#### Adicionar texto na seção Unattended-Upgrade::Origins-Pattern{:

 "origin=Raspbian,codename=${distro_codename},label=Raspbian"; </br>
 "origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";

 #### Descomentar e Configurar

 Unattended-Upgrade::AutoFixInterruptedDpkg "true";</br>
 Unattended-Upgrade::InstallOnShutdown "false";</br>
 Unattended-Upgrade::Mail "seuemail@seuprovedordeemail.com"; (descomentar e configurar apenas se for utilizar notificação por email)</br>
 Unattended-Upgrade::MailReport "on-change"; (always, only-on-error,on-change) (descomentar e configurar apenas se for utilizar notificação por email)</br>
 Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";</br>
 Unattended-Upgrade::Remove-Unused-Dependencies "true";</br>
 Unattended-Upgrade::Automatic-Reboot "true";</br>
 Unattended-Upgrade::Automatic-Reboot-Time "02:00";</br>
 Unattended-Upgrade::Verbose "false";</br>

### Configurar o envio de email

#### Configurar o arquivo /etc/postfix/sasl/sasl_passwd
>  [!TIP]
>  Se estiver utilizando o Gmail deverá criar uma Senha de App. Para essa opção ser habilitada é necessário ativar a Autenticação em duas Etapas (2 fatores).
>
>  Caso a pasta sasl em /etc/postfix/ não exista pode criá-la com o comando "sudo mkdir /etc/postfix/sasl/"

Criar arquivo sasl_passwd
```
  sudo touch /etc/postfix/sasl/sasl_passwd
```

Inserir conteúdo no arquivo sasl_passwd:
```
  sudo nano /etc/postfix/sasl/sasl_passwd
```

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

```
  sudo nano /etc/postfix/main.cf
```
##### Comentar a linha 

  #smtp_tls_security_level=may
  
##### Configurar o relay host

  relayhost = [servidor smtp]:587
  
 ##### Habilitar Autenticação SASL

  Inserir linhas no final do arquivo **main.cf**
  ```
  smtp_sasl_auth_enable = yes
  smtp_sasl_security_options = noanonymous
  smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
  smtp_tls_security_level = encrypt
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
### Gestão das atualizações

#### Configurar o intervalo das atualizações

```
  sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

Inserir este conteúdo no arquivo:

>[!NOTE]
> O número entre "aspas" indica o intevalo de dias em que as tarefas do APT serão realizadas. Esta configuração trabalha junto com a configuração de horário do APT (próximo tópico).

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
>[!IMPORTANT]
> O código OnCalendar= é necessário para realizar o override do horário original. Se não colocar este código a configuração do horário não irá funcionar.

Acrescentar o código
```
  [Timer]
  OnCalendar=
  OnCalendar=08:00 (coloar o horário)
  RandomizedDelaySec=0
```
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

>[!IMPORTANT]
> O código OnCalendar= é necessário para realizar o override do horário original. Se não colocar este código a configuração do horário não irá funcionar.

Acrescentar o código
```
  [Timer]
  OnCalendar=
  OnCalendar=08:20 (coloar o horário)
  RandomizedDelaySec=0
```
Reiniciar o serviço apt-daily-upgrade.timer
```
  sudo systemctl restart apt-daily-upgrade.timer
```
Verificar o agendamento
```
  sudo systemctl status apt-daily-upgrade.timer
```












  
