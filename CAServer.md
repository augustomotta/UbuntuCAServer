# Estrutura de diretórios da CA e dos Certificados Assinados (Linha )
/etc/ssl/             *{Diretório padrão das configurações do OpenSSL}*\
/etc/ssl/certs/       *{Diretório das CAs Confiáveis do Ubuntu Server}*\
/etc/ssl/crl/         *{Diretório de Requisição de Revogação de Certificados}*\
/etc/ssl/conf/        *{Diretório dos Arquivos de Configuração da CA e dos Certificados}*\
/etc/ssl/newcerts/    *{Diretório de Criação dos Novos Certificados Assinados}*\
/etc/ssl/private/     *{Diretório das Chaves Públicas e Privadas dos Certificados Assinados}*\
/etc/ssl/request/     *{Diretório das Requisições de Certificados Assinados}*


# Criar arquivo de banco de dados dos Certificados (Linha )
/etc/ssl/index.txt        *{Arquivo de Banco de Dados dos Certificados da CA}*\
/etc/ssl/index.txt.attr   *{Arquivo de Banco de Dados de Atributos dos Certificados da CA}*\
/etc/ssl/serial           *{Arquivo do Número de Série de Geração de Certificados}*\
/etc/ssl/conf/ca.conf     *{Arquivo de Configuração da Unidade Certificadora Raiz Confiável}*\


# Instalando a CA no sistema operacional Microsoft
```
 Pasta: Download\
   ca.crt (clicar duas vezes em cima do certificado)\
     Abrir\
       Instalar Certificado...\
         Assistente para Importação de Certificados\
           Máquina Local <Avançar>\
               Deseja permitir que este aplicativo faça alterações no seu dispositivo? <sim>\
                 Colocar todos os certificados no repositório a seguir\
                   Repositório de Certificados <Procurar>\
                 Autoridades de Certificação Raiz Confiáveis <OK>\
                 <Avançar>\
        <Concluir>\
     <OK>\
   <OK>\
```

```bash
sudo -i

mkdir -v /etc/ssl/{certs,conf,crl,newcerts,private,requests}

touch /etc/ssl/{index.txt,index.txt.attr,serial}

echo "1234" | sudo tee /etc/ssl/serial

echo "unique_subject = no" | sudo tee /etc/ssl/index.txt.attr

wget -v -O /etc/ssl/conf/ca.conf https://raw.githubusercontent.com/augustomotta/UbuntuCAServer/main/ca.conf

openssl genrsa -aes256 -out /etc/ssl/private/ca.key.old -passout pass:pass2k24 2048

openssl rsa -in /etc/ssl/private/ca.key.old -out /etc/ssl/private/ca.key -passin pass:pass2k24

rm -v /etc/ssl/private/ca.key.old

openssl rsa -noout -modulus -in /etc/ssl/private/ca.key | openssl md5

openssl req -new -sha256 -nodes -key /etc/ssl/private/ca.key -out /etc/ssl/requests/ca.csr -config /etc/ssl/conf/ca.conf

openssl req -copy_extensions copyall -new -x509 -sha256 -days 3650 -in /etc/ssl/requests/ca.csr -key /etc/ssl/private/ca.key -out /etc/ssl/newcerts/ca.crt -config /etc/ssl/conf/ca.conf

openssl x509 -noout -modulus -in /etc/ssl/newcerts/ca.crt | openssl md5

openssl x509 -noout -text -in /etc/ssl/newcerts/ca.crt

cp -v /etc/ssl/newcerts/ca.crt /usr/local/share/ca-certificates/

apt update

apt install ca-certificates

update-ca-certificates

mkdir -v /var/www/html/ca

cp -v /etc/ssl/newcerts/ca.crt /var/www/html/ca/
```


# Instalação no APACHE2
```bash
wget -v -O /etc/ssl/conf/apache2.conf https://raw.githubusercontent.com/augustomotta/UbuntuCAServer/main/apache2.conf
```

# Altere conforme sua necessidade
```bash
vim /etc/ssl/conf/apache2.conf
```

# Criação da chave
```bash
openssl genrsa -aes256 -out /etc/ssl/private/apache2.key.old -passout pass:pass2k24 2048

openssl rsa -in /etc/ssl/private/apache2.key.old -out /etc/ssl/private/apache2.key -passin pass:pass2k24

rm -v /etc/ssl/private/apache2.key.old

openssl rsa -noout -modulus -in /etc/ssl/private/apache2.key | openssl md5

openssl req -new -sha256 -nodes -key /etc/ssl/private/apache2.key -out /etc/ssl/requests/apache2.csr -extensions v3_req -config /etc/ssl/conf/apache2.conf

openssl ca -in /etc/ssl/requests/apache2.csr -out /etc/ssl/newcerts/apache2.crt -config /etc/ssl/conf/ca.conf -extensions v3_req -extfile /etc/ssl/conf/apache2.conf

openssl x509 -noout -modulus -in /etc/ssl/newcerts/apache2.crt | openssl md5

openssl x509 -noout -text -in /etc/ssl/newcerts/apache2.crt

cat -n /etc/ssl/index.txt

cat -n /etc/ssl/serial

wget -v -O /etc/apache2/sites-available/default-ssl.conf https://raw.githubusercontent.com/augustomotta/UbuntuCAServer/main/default.ssl.conf
```

# Altere conforme sua necessidade
```bash
vim /etc/apache2/sites-available/default-ssl.conf
```

# Altere o arquivo /etc/apache2/ports.conf
```bash
vim /etc/apache2/ports.conf
```

```
   <IfModule ssl_module>
	  Listen 443
	  </IfModule>
	
	  <IfModule mod_gnutls.c>
		  Listen 443
	  </IfModule>
```

```bash
a2enmod ssl headers

apache2ctl configtest

a2ensite default-ssl

systemctl restart apache2

systemctl status apache2

journalctl -xeu apache2

lsof -nP -iTCP:'80,443' -sTCP:LISTEN

echo | openssl s_client -connect localhost:443 -servername 127.0.0.1 -showcerts
```

# Desativar site padrão Apache2
```bash
a2dissite 000-default.conf
```