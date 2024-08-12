#### Automação de atualizações do sistema Raspbian (debian bookworm) no Raspberry Pi e envio de notificação por email com autenticação SASL.

#### pacotes necessários
  sudo apt install unattended-upgrades</br>
  sudo apt install postfix</br>
  sudo apt install libsasl2-modules

#### criar banco de dados passwd SASL (depois de configurar o arquivo sasl_passwd)
  sudo postmap /etc/postfix/sasl/sasl_passwd
  
#### definir permissões de arquivos
  sudo chown root:root /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db</br>
  sudo chmod 0600 /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db

Configurar arquivo main.cf e reiniciar postfix (sudo systemctl reload postfix.service)

#### testar envio email
  sudo unattended-upgrades -d
