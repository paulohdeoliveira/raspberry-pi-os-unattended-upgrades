#### Automação de atualizações do sistema Raspbian (debian bookworm) no Raspberry Pi e envio de notificação por email com autenticação SASL.

#### Pacotes necessários
```
  sudo apt install unattended-upgrades
  sudo apt install postfix
  sudo apt install libsasl2-modules
```

#### Configurar o arquivo /etc/postfix/sasl/sasl_passwd ####
>  [!NOTE]
>  Se estiver utilizando o Gmail deverá criar uma Senha de App. Para essa opção ser habilitada é necessário ativar a Autenticação em duas Etapas (2 fatores).
```
[servidor smtp]:587 seuemail:suasenha
```

#### Criar banco de dados passwd SASL ####
> [!IMPORTANT]
> Executar o comando abaixo sempre que alterar o arquivo sasl_passwd
```
  sudo postmap /etc/postfix/sasl/sasl_passwd
```
#### Definir permissões de arquivos ####
```
  sudo chown root:root /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db
  sudo chmod 0600 /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db
```
#### Configurar arquivo o main.cf ####

```
  sudo nano /etc/postfix/main.cf
```
##### Comentar a linha ##### 

  #smtp_tls_security_level=may
  
##### Configurar o relay host #####

  relayhost = [servidor smtp]:587
  
 ##### Habilitar Autenticação SASL #####

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

#### Testar o envio de email ####
```
  sudo unattended-upgrades -d
```
 #### Configurar o horário das atualizações ####

>[!TIP]
> Não utilize o Cron para agendar as atualizações. O Unattended-Upgrades utiliza a configuração de horário do APT!

Caminho dos arquivos de horário de atualização APT:

**Horário de download**

/lib/systemd/system/apt-daily.timer

**Horário de upgrade**

/lib/systemd/system/apt-daily-upgrade.timer

**Para definir o horário de Download**
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
**Para definir o horário de Upgrade**
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












  
