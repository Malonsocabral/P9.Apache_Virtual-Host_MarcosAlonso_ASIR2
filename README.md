# P9.Apache_Virtual-Host_MarcosAlonso_ASIR2

 ## Utiliza o creado ata agora en docker. Crea un Readme.md para ir documentando os pasos. Engade un DNS ó docker-compose (ou o que xa tes)

    Usa docker-compose para configura-las IP fixas ós dous contenedores

    O DNS ten que resolver dous dominios á la ip do apache, por exemplo:
        www.fabulasoscuras.com
        www.fabulasmaravillosas.com
    Proba a utiliza-la directiva DirectoryIndex
    Configura dous virtual-host separados para cada dominio no mesmo porto (80)

## Apache
Primeiro de todo debemos facer o .yaml, polo que comenzaremos explicando as primeiras liñas de este yaml:
```services:
  # Servizo 'web' que executa un servidor web con PHP e Apache
  web:
    image: php:7.4-apache   # Imaxe oficial de PHP con Apache, versión 7.4
    container_name: P-Apache  # Nome do contedor, 'P-Apache'
    ports:
      - "80:80"   # Mapea o porto 80 do contedor ao porto 80 do host (HTTP)
    volumes:
      - ./www:/var/www   # Mapea o directorio 'www' do host ao directorio '/var/www' do contedor (para arquivos web)
      - ./confApache:/etc/apache2   # Mapea o directorio de configuración de Apache do host ao do contedor
    networks:
      redp9:   # Conéctase á rede 'redp9'
        ipv4_address: 172.19.4.2   # Asigna a IP estática 172.19.4.2 dentro da rede 'redp9'
```
A modo de resumo, como podemos ver, configuramos o servidor apache no porto 80 e conectado a unha red chamada `p9red`, e ademais importamos uns volumes que temos nos xa creados a modo de configuracion como poden ser as paxinas html que explicarei mas tarde.  

## Bind9 (DNS)
O proximo que hay no yaml e o servidor dns que expliquei no yaml nas seguintes liñas: 
``` # Servizo 'dns' que executa un servidor DNS con Bind9
  dns:
    container_name: dns-bind9   # Nome do contedor, 'dns-bind9'
    image: ubuntu/bind9   # Imaxe oficial de Ubuntu con Bind9 (servidor DNS)
    ports:
      - "57:53"   # Mapea o porto 53 do contedor ao porto 57 do host (para evitar conflitos co porto 53 do host)
    volumes:
      - ./confDNS/conf:/etc/bind   # Mapea o directorio de configuración de DNS do host ao contedor
      - ./confDNS/zonas:/var/lib/bind   # Mapea o directorio de zonas DNS do host ao contedor
    networks:
      redp9:   # Conéctase á rede 'redp9'
        ipv4_address: 172.19.4.3   # Asigna a IP estática 172.19.4.3 dentro da rede 'redp9'
```  
E como podemos observar nesta explicacion, se montan varios volumes que mapean e configuran os distintos dominios e zonas que teremos neste servidor, ademais o lanzamos no port 51 xa que no meu caso, non da ningun erro.  
## Creacion de red   
E por ultimo, podemos ver que no yaml configuramos una red que chamase `redp9` no que creamos e configuramos a subred, aunque a explicacion mais exacta adjuntoa a continuacion:  
``` 
# Definición das redes que se usan polos servizos
networks:
  redp9:   # Nome da rede 'redp9'
    driver: bridge   # Usa o controlador 'bridge' para crear unha rede privada e illada
    ipam:   # Configuración da asignación de IPs
      driver: default   # Usa o controlador predeterminado de IPs
      config:
        - subnet: 172.19.0.0/16   # Define a subrede 172.19.0.0/16 para a rede 'redp9'
```   
## Configuramos apache (arquivo confApache)
No meu caso creei esta parte a partir dun repositorio de damian o cal deixo o link a continuacion : https://github.com/damiancastelao/ApacheDosDominios   
Unha vez todo descargado , podemos meter os nosos dous arquivos na carpeta `sites-avaliable` e `sites-enabled`, cabe destacar que os dous arquivos (fabulasmarabillosas e fabulasoscuras) temos que ponerlo si o si na cartepa de "sites-enable" para que pueda funcionar.  
Logo tambien explicarei algunhas configuracions que puxen como o `Documentroot` que e donde se atopa o html, ou o alias e o name do dominio que no meu caso e "www.fabulasmaravillosas.asircastelao.int" e "www.fabulasmaravillosas.asircastelao.int" respectivamente.  
  
  Ademais se queremos empregar varios portos, debemos meternos no ficheiro `ports.conf` e engadir una liña como por exemplo `listen 8083` ou o porto que quiseramos configurar.  
## Configuracions en apache da web fabulasmaravillosas (arquivo fabulasmaravillosas.conf)  
Para esto modificaremos o arquivo `fabulasmaravillosas.conf` dentro da configuracion de `sites-enabled`. Eo primeiro de todo , podemos ver que se chamara ao porto 80, con un correo webmaster@localhost, con un servidor de nombre "fabulasmaravillosas.asircastelao.int",etc. Todo estas configuracion as podemos observar a continuacion:


```<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasmaravillosas.asircastelao.int
    ServerAlias www.fabulasmaravillosas.asircastelao.int
    DocumentRoot /var/www/fabulasmaravillosas
</VirtualHost>
```  
## Configuracions en apache da web fabulasoscuras (arquivo fabulasoscuras.conf)
Este punto e o mesmo que o anterior polo que solo deixarei o codigo de configuracion para que se pode apreciar todo:  
```<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasoscuras.asircastelao.int
    ServerAlias www.fabulasoscuras.asircastelao.int
    DocumentRoot /var/www/fabulasoscuras
</VirtualHost>
```
## Configurando o DNS (ficheiro confDNS)  
Pimeiro de todo, esta carpeta ten so dous suscarpetas onde configuramos nunha a configuracion propia do DNS e na outra as zonas que tera ese DNS.  
### Configuracion das zonas (arquivo zonas)
Primeiro , cando entramos nesta carpeta podemos ver que el fichero chamase "db.asircastelao(que e o nome da zona).int", e este arquivo ten configuracions propias da zona que explicarei a continuacion: 
```$TTL    604800   # Establece o tempo de vida (TTL) predeterminado de todos os rexistros en segundos. Neste caso, 604800 segundos, que son 7 días.

# A seguinte liña define a zona SOA (Start of Authority), que indica a información principal sobre a zona DNS
@       IN      SOA     ns.asircastelao.int. some.email.address.com. (
                              2         ; Serial   # O número de serie do rexistro SOA. Debería incrementarse con cada cambio no arquivo.
                         604800         ; Refresh  # O tempo en segundos (604800 = 7 días) que un servidor escravo debe esperar antes de comprobar se a zona se actualizou.
                          86400         ; Retry    # O tempo en segundos (86400 = 1 día) que un servidor escravo debe esperar antes de reintentar se non pode obter a zona primaria.
                        2419200         ; Expire   # O tempo en segundos (2419200 = 4 semanas) tras o cal un servidor escravo debe eliminar a súa copia da zona se non pode contactarse co servidor mestre.
                         604800 )       ; Negative Cache TTL # O tempo en segundos (604800 = 7 días) que un servidor debe gardar en caché unha resposta negativa (es decir, se non existe un rexistro).

# Aquí especificamos que o servidor de nomes principal para esta zona é ns.asircastelao.int.
@       IN      NS      ns.asircastelao.int.

# Definimos a dirección IP asociada ao nome do servidor de nomes ns.asircastelao.int.
ns      IN      A       172.19.4.3  # O rexistro 'A' asigna a IP 172.19.4.3 ao nome 'ns'.

# Definimos a dirección IP para o dominio 'fabulasoscuras.asircastelao.int'.
fabulasoscuras       IN      A       172.19.4.2  # O rexistro 'A' asigna a IP 172.19.4.2 ao nome 'fabulasoscuras'.

# Definimos a dirección IP para o dominio 'fabulasmaravillosas.asircastelao.int'.
fabulasmaravillosas     IN      A       172.19.4.2  # O rexistro 'A' asigna a IP 172.19.4.2 ao nome 'fabulasmaravillosas'.
```
### Configuracion propia del DNS (arquivo conf)
Para comezar, neste ficheiro debemos de tener 3 ficheiros que teñes una configuracion moy sinxela, o primeiro que vou explicar e o arquivo `named.conf.local` que o que fai e chamar ao arquivo que acabo de explicar da zona indicando o tipo e a ruta desta zona:
```
zone "asircastelao.int" {
    type master;
    file "/var/lib/bind/db.asircastelao.int";
    allow-query {
    	any;
    	};
};
```

Continuando co ficheiro `named.conf.options` , aqui e onde configuramos os servidores forwarders como poden ser google ou cloudflare, seguido de outras configuracions como la resolucion recursiva que explico a continuacion :

```options {
    directory "/var/cache/bind";
    recursion yes;                      # Permitir la resolución recursiva
    allow-query { any; };               # Permitir consultas desde cualquier IP
    dnssec-validation no;               #Para que non verifique o noso dns e non dea fallo
    forwarders {
        8.8.8.8;                        # Google DNS
        1.1.1.1;                        # Cloudflare DNS
    };
    listen-on { any; };                 # Escuchar en todas las interfaces
    listen-on-v6 { any; };
};
```    
E para rematar con esta carpeta, podemos ver o ficheiro `named.conf` onde ten so dous liñas de codigo que enlazan os dous ficheiros que acabamos de explicar nesta seccion:  
```include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```

Por ultimo, unha cousa a destacar e que copiei do repo de damian as configuracions principais das zonas inversas como se pode ver no ficheiro named.conf.default-zones como se pode ver a continuacion:   
```// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};
```

## Configurando las paginas (ficheiro www)  
Para configurar este volumen que nombramos no yaml e no apache, debemos crear cada carpeta con su respectivo html ya que se esta ubicacion es de donde va a coger el servidor apache para mostrar las paginas web.  

Polo que neste caso, so debemos crear as carpetas fabulasmarabillosas e fabulasescuras con un fichero index.html co contenido da paxina. Para ver este apartado , non vou pegar o codigo xa que e longo e é mis facil ir no repositorio a carpeta www para observar mellor ambas paxinas.

## Configuracions externas ao .yaml  
Primeiro de todo, para que funcione o noso servidor DNS e non de ningun erro, debemos irnos a configuracion da propia maquina virtual, e nas opcions de red, debemos cambiar a NAT que ven por defecto a `Adaptador puente` e logo darlle a premitir todo no modo promiscuo.  

Para que cando hagamos o `docker compose up` funcione todo correctamente, debemos modificar el fichero resolved.conf no sistema operativo. E para esto debemos facer un `sudo nano /etc/systemd/resolved.conf` e  buscar a liña donde pon DNS e poñer `DNS=<ip servidor>#<puerto servidor>` que no meu caso é `DNS=172.19.4.3#57` logo facemos Ctrl + S y Ctrl + X para salir y guardar.  
E para que estos cambios se apliquen debemos facer un `sudo systemctl restart systemd-resolved`.

Unha vez feito, entramos na configuración de red da nosa máquina ubuntu. E no meu caso so teño que ir arriba a dereita, logo entramos na red, e entramos na ruedita tipica das configuracions.  
Unha vez aqui, damoslle ao apartado de IPv4 e desactivamos a opción de DNS automático  
Para que estos cambios se apliquen debemos reiniciar a maquina ubuntu.  

  E por ultimo para comprobar que funciona debemos meternos nunha consola , facer o `docker compose up` na carpeta da practica e comprobamos que non da ningun erro e vamos a o navegador e buscamos por url as nosas paxinas, que no meu caso son `www.fabulasmaravillosas.asircastelao.int` e `www.fabulasoscuras.asircastelao.int`.
