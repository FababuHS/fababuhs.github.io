# IAW-PRACTICA-JEKYLL
Álvaro Alejandro Fababú López  
Creación de blogs con Jekyll y GitHub Pages  
2º ASIR - IES Celia Viñas
#
### Jekyll es la herramienta de GitHub que permite la creación de sitios web estáticos en forma de blog o sitio web personal, basándose en el contenido de un repositorio de GitHub.
*** 
## Creación del proyecto
Lo primero que tenemos que hacer es crear un directorio en nuestro equipo anfitrión, desde dónde haremos correr el contenedor de Jekyll con el comando
~~~
docker run -it --rm -v "$PWD:/srv/jekyll" jekyll/jekyll jekyll
~~~
Una vez tengamos el contenedor corriendo, el siguiente paso es la creación de la estructura de directorios y los archivos base de un proyecto Jekyll. Esto lo hacemos mediante el comando
~~~
docker run -it --rm -v "$PWD:/srv/jekyll" jekyll/jekyll jekyll new blog
~~~
***
## Archivo de configuración _config.yml
Este archivo es desde dónde podremos realizar la personalización del índice de la página y del estilo del sitio. 
~~~
title: Página de IAW de Álvaro Fababú
email: fabluck9@gmail.com
description: >- # this means to ignore newlines until "baseurl:"
  Write an awesome description for your new site here. You can edit this
  line in _config.yml. It will appear in your document head meta (for
  Google search results) and in your feed.xml site description.
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
twitter_username: fababux
github_username:  FababuHS

# Build settings
theme: minima
plugins:
  - jekyll-feed
~~~
Nos permite configurar el título de la página así como el texto descriptivo que queremos que aparezca y el email asociado al site. También podremos indicar nuestro usuario tanto de Twitter como de GitHub, así como especificar el tema que queramos utilizar y los plugins.
***
## Población del sitio
Jekyll permite la creación de posts y de páginas. Los posts se encuentran contenidos dentro del directorio *_posts*, y su nombre debe seguir la nomenclatura *yyyy-mm-dd-nombredelpost.md*. Inicialmente habrá un post creado por defecto y podemos añadir los que queramos. Los posts que creemos deberán comenzar con la siguiente cabecera:
~~~
---
layout: post
title:  "Welcome to Jekyll!"
date:   2022-03-07 05:17:55 -0600
categories: jekyll update
---
~~~
En la directiva *layout* indicamos que se trata de un post, en *title* pondremos el título que queramos ponerle al post para que se muestre en el sitio y podremos indicar las categorías relacionadas con el post en *categories*. Finalmente, la directiva *date* contiene la fecha y hora en la que el post se publicará en el site, por lo que si introducimos una fecha futura el post no aparecerá automáticamente, si no que se desbloqueará en el momento indicado.  
En el caso de las páginas, también se trata de archivos en formato Markdown, pero colgarán directamente del raíz del proyecto en vez de alojarse dentro de otro directorio. Igual que en el caso de los posts, las páginas deben comenzar por una cabecera similar a la siguiente:
~~~
---
layout: page
title: Prueba
permalink: /prueba/
---
~~~
Donde dentro de la directiva *permalink* indicaremos el alias con el que aparecerá a la hora de acceder a esta. Es importante mencionar que aunque las páginas se creen sobre ficheros en formato Markdown, estas pueden contener código HTML que Jekyll se encargará de transformar.
