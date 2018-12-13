# Cómo usar Home Assistant fuera de la red usando el protocolo https gracias a Apache y Cloudflare

Para realizar esto, necesitamos algunas cosas antes de empezar:
1. Un dominio, si no tienes uno, no hace falta que pagues, existe http://freenom.com que te ofrece dominios gratuitos durante 1 año, sin necesidad de tarjeta ni nada, tan solo tu correo electrónico.
2. Una cuenta de CloudFlare, gracias a esta empresa y su cuenta gratuita, podremos tener un certificado https a nivel de DNS, es decir, no hará falta instalar nada en nuestro servidor.
3. Home Assistant instalado, no es compatible con Hass.io ya que éste no tiene Apache instalado.
4. Una IP fija/estática. Como muchos de vosotros (por no decir todos) no teneis IP estática, os recomiendo [usar mi script de python](https://github.com/angelsocias/CloudFlare-IP-Changer-Bot) que, gracias a nuestro dominio gratuito y CloudFlare mantendremos nuestro dominio apuntando siempre a nuestra IP pública. De lo contrario, habrá que cambiar la IP manualmemnte cada vez que cambie.


Una vez tenemos todo, nuestro dominio apuntando a nuestra IP desde CloudFlare, procedemos a configurar nuestro servidor.

1. Primero de todo necesitamos tener Apache instalado, para ello, ejecutamos el siguiente comando: ```sudo apt-get install apache2```
2. Una vez instalado, debemos activar algunos "plugins" para que posteriormente funcione todo correctamente. Ejecutamos los siguientes comandos:
  * ```sudo a2enmod proxy```
  * ```sudo a2enmod proxy_http```
  * ```sudo a2enmod proxy_balancer```
  * ```sudo a2enmod lbmethod_byrequests```
  
Y reiniciamos el servicio para que se apliquen los cambios.
  * ```sudo service apache2 reload```
  
3. Una vez tenemos instalado apache y activados los mods necesarios, vamos a configurar el VirtualHost, digamos que esto es lo que va a hacer que cuando entremos por nuestro dominio, redirija internamente a nuestra instalación de HomeAssistant.
Para ello, vamos a crear primero el archivo, con el siguiente comando:
  * ```sudo nano /etc/apache2/sites-available/homeassistant.conf```

Se nos abrirá una ventana, esto es un editor de texto en terminal. Aquí pegaremos el siguiente trozo de código (Para pegarlo, si usas el cliente Putty, se hace haciendo click derecho):
```
  <VirtualHost *:80>
  ProxyPreserveHost On
  ProxyRequests Off
  ServerName dominio.com

  ProxyPass "/" "http://localhost:8123/"
  ProxyPassReverse "/" "http://localhost:8123/"

  RewriteEngine on
  RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]
  RewriteCond %{HTTP:CONNECTION} ^Upgrade$ [NC]
  RewriteRule .* ws://localhost:8123%{REQUEST_URI} [P]
</VirtualHost>
```

Donde pone dominio.com, debemos cambiarlo por el dominio creado con FreeNom.

Una vez pegado y cambiado el dominio, presionamos teclas Ctrl+O, Enter para guardar el archivo y luego Ctrl+X para cerrarlo.

Ahora debemos activar el VirtualHost, se hace con el siguiente comando:
  * ```sudo a2ensite homeassistant.conf```
  
Y reiniciamos apache2 para que haga efecto el cambio:
  * ```sudo service apache2 restart```

Si hemos realizado todos los pasos correctamente, ya deberiamos poder entrar desde nuestro dominio que hemos creado, a HomeAssistant, pudiendo usar https si hemos puesto el registro en CloudFlare con la nube en naranja.
