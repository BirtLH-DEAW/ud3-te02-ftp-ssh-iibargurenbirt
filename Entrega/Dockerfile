# Primer paso. Clonar repo privado
FROM ubuntu:24.04 as clientegit

# Copiar clave privada y ajustar permisos
COPY ssh/id_rsa /id_rsa
RUN chmod 600 /id_rsa

# Instalar cliente git
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y git

#Clonar repositorio
RUN eval $(ssh-agent) && \
	ssh-add /id_rsa && \
	ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts && \
	git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /UD3-ftp_anonimo

# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:24.04

# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host" \
    maintainer="usuarioBIRT <iibarguren@birt.eus>"

# Actualizamos la lista de paquetes e instalamos nano, apache2, ssh y proftpd
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 proftpd proftpd-mod-crypto openssh-server && \
    rm -rf /var/lib/apt/lists/*

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioprimero /var/www/html/sitiosegundo

# Creamos directorio para el servidor ftp anonimo
RUN mkdir -p /srv/anonymous/
COPY --from=clientegit /UD3-ftp_anonimo/ /srv/anonymous/

# Copiamos archivos de apache al contenedor
COPY apache/indexPrimero.html apache/indexSegundo.html apache/sitioPrimero.conf apache/sitioSegundo.conf apache/sitioPrimero.key apache/SitioPrimero.cer /

# Movemos los archivos a sus ubicaciones adecuadas
RUN mv /indexPrimero.html /var/www/html/sitioprimero/index.html && \
    mv /indexSegundo.html /var/www/html/sitiosegundo/index.html && \
    mv /sitioPrimero.conf /etc/apache2/sites-available/sitio1.conf && \
    mv /sitioSegundo.conf /etc/apache2/sites-available/sitio2.conf && \
    mv /sitioPrimero.key /etc/ssl/private/sitio1.key && \
    mv /SitioPrimero.cer /etc/ssl/certs/sitio1.cer

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitio1.conf && \
    a2ensite sitio2.conf && \
    a2enmod ssl
    
# Copiamos archivos de proftpd al contenedor
COPY ftp/proftpd.conf ftp/modules.conf ftp/tls.conf ftp/proftpd.crt ftp/proftpd.key /

# Movemos los ficheros a sus ubicaciones
RUN	mv /proftpd.conf /etc/proftpd/ && \
	mv /modules.conf /etc/proftpd/ && \
	mv /tls.conf /etc/proftpd/ && \
	mv /proftpd.crt /etc/ssl/certs/ && \
	mv /proftpd.key /etc/ssl/private/


# Creamos los usuarios del sistema, les ponemos contraseña y ajustamos permisos de sus home
RUN groupadd deaw
RUN useradd iibarguren1 -p iibarguren1 -c -m -d /var/www/html/sitioprimero -s /sbin/nologin -g deaw
RUN useradd iibarguren2 -p iibarguren2 -c -m -d /var/www/html/sitiosegundo -s /bin/bash -g deaw

RUN echo "iibarguren1:iibarguren1" | chpasswd
RUN echo "iibarguren2:iibarguren2" | chpasswd

RUN chown -R iibarguren1 /var/www/html/sitioprimero
RUN chmod -R u+w /var/www/html/sitioprimero

RUN chown -R iibarguren2 /var/www/html/sitiosegundo
RUN chmod -R u+w /var/www/html/sitiosegundo


# Exponemos los puertos
EXPOSE 80
EXPOSE 443

EXPOSE 21
EXPOSE 50000-50030

EXPOSE 22

# Comando por defecto al iniciar el contenedor, se inician los servicios de ftp y ssh y se inicia apache en el foreground para que el contenedor no termine su ejecucion
CMD ["/bin/sh", "-c" , "service proftpd start ; service ssh start ; apachectl -D FOREGROUND"]
