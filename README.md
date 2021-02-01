# LDAP
Autenticação LDAP / Básico / Testes / Centos 7 / Server Local


1. yum install openldap openldap-clients openldap-servers -y # Instalar pacotes

systemctl enable slapd # Daemon do LDAP
systemctl status slapd # Verificar se está rodando

ss -ln | grep 389 # Ver se a porta está escutando

firewall-cmd --state # Ver se firewall está rodando 

firewall --add-service=ldap --permanent # Caso seja necessario, adicione o ldap no firewall
firewall-cmd --reload 

slappasswd # Criar senha para o root /  Guarde o HASH.


##############-------Aplicando instruções arquivos .ldif---------###############

2. Arquivos prontos base.ldif db.ldif monitor.ldif #Podendo ser editado conforme a sua necessidade

vi db.ldif #Editando arquivo
Pegue a HASH que você guardou e inclua no seguinte campo - >  olcRootPW: {HASH}

ldapmodify -H ldapi:/// -f db.ldif #Aplicar arquivo de instruções

ldapmodify -H ldapi:/// -f monitor.ldif 

cd /usr/share/openldap-servers/

cat DB_EXAMPLE # Arquivo de exemplo de configuração para o Banco de Dados, para ver sobre escalonamento, desempenho, tunning -visite >>https://www.openldap.org/doc/admin24/

cp DB_CONFIG.example /var/lib/ldap/DB_CONFIG # Copiar arquivo

cd /var/lib/ldap/ #Acesse o diretório

chown ldap:ldap DB_CONFIG #Trocar o grupo do arquivo para ldap

############------Aplicando schemas--------#################

Vá até >> /etc/openldap/schema/

ldapadd -H ldapi:/// -f cosine.ldif # Aplicar schema

ldapadd -H ldapi:/// -f inetorgperson.ldif

ldapadd -H ldapi:/// -f nis.lidf

 #Adicionando estrutura 
 ldapadd -x -W -D "cn=ldapadm,dc=exemplo,dc=com" -f base.ldif # -x (Autenticação Simples) -W (Segurança) -D (dn do usuario root) 
 
ldappasswd -S -W -D "cn=ldapadm,dc=exemplo,dc=com" -x "uid=teste,ou=Lojas,o=empresa,c=BR,dc=exemplo,dc=com"  #Adicionando a senha para o usuário teste
 
ldapsearch -x cn=joao -b dc=4fasters,dc=com # Veja o usuário criado (teste) e sua senha

#############################------------Configuração do LDAP Client----------#######################################

Usando debian para o cliente

ldapsearch -h <ipdoseuservidor> -x -b O=* -B c=BR,dc=exemplo,dc=com # Buscar no ldap ->> Caso não consiga veja sua configurações de firewall/rede
  
  #Instale a libpam
apt install libpam-ldap libnss-ldap  -y
  
  Configurando libnss-ldap
  
  url do servidor ldap://<ipdoservidorldap>
  
  o nome especifico da base de pesquisa: dc=exemplo,dc=com
  
  Versão->> 3
  
  conta ldap para o root: cn=ldapadm,dc=exemplo,dc=com
  
  Senha: <senhadoldapadm>
  
  nsswitch.conf <Ok>
  
  Permitir que a conta admin do ldap se comporte como usuário root local <Sim>

  A base de dados do ldap requer autenticação <Não>
  
  infomre a conta admin do ldap ->> cn=ldapadm,dc=exemplo,dc=com
  senha: <senhaldpadadm>
  
  vi /etc/nsswitch.conf # Editando, definir ordem
  
  
 no campo passwd: compat ldap
 no campo group: compat ldap
 no campo shadow: compat ldap
  
vá até cd /etc/pam.d

vi common-session # Editar arquivo

adicione ->> session required        pammkhomedir.so skel=/etc/skel umask=077

salve e reinicie

iniciar sessão> usuário: teste senha: teste

abra o terminal e digite getent 
e veja as informações sobre o usuário criado no ldap





