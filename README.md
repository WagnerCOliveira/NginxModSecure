# NginxModSecure

## Criar Server Proxy Mod Secure Homologação

### Instalar Nginx e ModSecurity 3.0

Instalar Nginx e ModSecurity 3.0

Instalação Centos 7 Minimal dynamic
Change SELinux to permissive
Criando o arquivo do repositório para usar a versão mais estável do nginx

Atualização do servidor

```
#yum update
```

Instalação do Nginx, caso não tenha o repositório do nginx no servidor instale o repositório primeiro.

```
#yum install nginx -y
```

Liberando http e https no Iptables

```
#firewall-cmd --zone=public --permanent --add-service=http
#firewall-cmd --zone=public --permanent --add-service=https
#firewall-cmd --reload
```

Check nginx -v for the NGINX version and change appropriately;

Instalando as dependências

Download e Instalação ModSecurity

```
#git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity
#cd ModSecurity
#git submodule init
#git submodule update
#./build.sh
#./configure
#make
#make install
#yum groupinstall 'Development Tools' -y
#yum install gcc-c++ flex bison yajl yajl-devel curl-devel curl GeoIP-devel.x86_64 doxygen zlib-devel pcre-devel.x86_64 -y
#yum install lmdb lmdb-devel libxml2 libxml2-devel ssdeep ssdeep-devel lua lua-devel wget vim -y

```

Instalação do module dinâmico entre o ModSecurity e Nginx

```
git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```

Execute nginx -v para baixar a versão que foi instalada

```
#wget http://nginx.org/download/nginx-1.14.2.tar.gz
#tar zxvf nginx-1.14.2.tar.gz
#cd nginx-1.14.2
#./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
#make modules
#cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules
#mkdir /etc/nginx/modsec
#cp modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf
#cp ~/ModSecurity/unicode.mapping /etc/nginx/modsec/
```

Em seguida, adicione a instrução load_module ao nginx.conf (/etc/nginx/) no contexto principal (nível superior):

load_module modules/ngx_http_modsecurity_module.so;


Habilitando o OWASP CRS
Baixe o OWASP CRS mais recente do GitHub e extraia as regras em (/usr/local) ou em outro local do seu
escolha.

```
wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v3.0.2.tar.gz

#tar -xzvf v3.0.2.tar.gz
#mv owasp-modsecurity-crs-3.0.2 /usr/local
```

Crie o arquivo (crs‑setup.conf) como uma cópia de crs‑setup.conf.example.

```
#cd /usr/local/owasp-modsecurity-crs-3.0.2
#cp crs-setup.conf.example crs-setup.conf
```

Crie um arquivo chamado (main.conf) e adicione as diretivas de inclusão no arquivo de configuração principal do NGINX WAF
(/etc/nginx/modsec/main.conf) para ler a configuração e as regras do CRS. Comente quaisquer outras regras que
pode já existir no arquivo, como a diretiva SecRule de amostra criada nessa seção.


```
#Include the recommended configuration
Include /etc/nginx/modsec/modsecurity.conf

#OWASP CRS v3 rules
Include /usr/local/owasp-modsecurity-crs-3.0.2/crs-setup.conf
#Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-901-INITIALIZATION.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-905-COMMON-EXCEPTIONS.conf
#Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-910-IP-REPUTATION.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-911-METHOD-ENFORCEMENT.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-912-DOS-PROTECTION.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-913-SCANNER-DETECTION.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-921-PROTOCOL-ATTACK.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-931-APPLICATION-ATTACK-RFI.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-933-APPLICATION-ATTACK-PHP.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-949-BLOCKING-EVALUATION.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-950-DATA-LEAKAGES.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-951-DATA-LEAKAGES-SQL.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-952-DATA-LEAKAGES-JAVA.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-953-DATA-LEAKAGES-PHP.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-954-DATA-LEAKAGES-IIS.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-959-BLOCKING-EVALUATION.conf
Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-980-CORRELATION.conf
#Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example
```

Altere a diretiva SecRuleEngine na configuração para alterar do modo padrão "apenas detecção" para
eliminando ativamente o tráfego malicioso.

```
#sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/nginx/modsec/modsecurity.conf
```
Adicione as diretivas modsecurity e modsecurity_rules_file à configuração NGINX para habilitar
ModSecurity:

```
server {
# ...
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/main.conf;
}

```

Inicie o NGINX e habilite o serviço na inicialização.

```
#systemctl start nginx
#systemctl enable nginx
```

Referências:

* https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/
* https://www.nginx.com/blog/compiling-and-installing-modsecurity-for-open-source-nginx/
* https://docs.nginx.com/nginx-waf/admin-guide/nginx-plus-modsecurity-waf-owasp-crs/
* https://github.com/SpiderLabs/ModSecurity/wiki/Compilation-recipes-for-v3.x#centos-7-minimal

