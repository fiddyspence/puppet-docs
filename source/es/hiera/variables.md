---
layout: default
title: "Hiera 1: Variables e Interpolación"
canonical: "/es/hiera/variables.html"
---

Hiera recibe un conjunto de variables cada vez que sea invocado, y el [archivo de configuración](http://docs.puppetlabs.com/hiera/1/configuring.html) y [las fuentes de información](http://docs.puppetlabs.com/hiera/1/data_sources.html) pueden insertar estas variables en la configuración y en los datos. Esto te permite generar fuentes de información dinámicas en la [jerarquía](http://docs.puppetlabs.com/hiera/1/hierarchy.html), y evitar repeticiones cuando escribes datos.

## Insertar valores de variables

Los *símbolos de interpolación* se ven así: *%{variable}*, un signo de porcentaje seguido de un par de llaves que contienen el nombre de la variable.

Si alguna [opción en el archivo de configuración](http://docs.puppetlabs.com/hiera/1/configuring.html) o [valor en una fuente de información](http://docs.puppetlabs.com/hiera/1/data_sources.html) contiene un símbolo de interpolación, Hiera lo remplazará con el valor de la variable. Los símbolos de interpolación pueden aparecer solos o como parte de un string.

+ Hiera sólo puede interpolar variables cuyos valores sean **strings**; los **números** de Puppet también pasan como strings y se pueden utilizar de forma segura. No puedes interpolar variables cuyos valores sean booleanos, números que no sean de Puppet, arrays, hashes, referencias a recursos, o un valor **undef** (*indefinido*) explícito.
+ Además, Hiera no puede interpolar un **elemento** individual de ningún array o hash, incluso si el valor de este elemento es un string.

### En fuentes de información

El uso principal de la interpolación es en el [archivo de configuración](http://docs.puppetlabs.com/hiera/1/configuring.html), donde puedes establecer fuentes información dinámicas en la [jerarquía](http://docs.puppetlabs.com/hiera/1/hierarchy.html):

	---
	:hierarchy:
	- %{::clientcert}
	- %{::custom_location}
	- virtual_%{::is_virtual}
	- %{::environment}
	- common

En este ejemplo, cada fuente de información excepto la última, variarán dependiendo de los valores actuales de las variables **::clientcert, ::custom_location, ::is_virtual,** y **::enviroment**.

### En otras opciones de configuración

También puedes interpolar variables dentro de otras [opciones de configuración](http://docs.puppetlabs.com/hiera/1/configuring.html), como **:datadir** (en los backends YAML y JSON):

	:yaml:
  	  :datadir: /etc/puppet/hieradata/%{::environment}

Este ejemplo te dejaría utilizar directorios de información completamente separados para tus entornos de producción y de desarrollo.

### En datos

Dentro de una fuente de información, puedes interpolar variables en cualquier string, ya sea un valor aislado o parte de un hash o array. Esto puede ser útil para valores que deberían ser diferentes para cada nodo, pero que difieren **de forma previsible**

	# /var/lib/hiera/common.yaml
	---
	smtpserver: mail.%{::domain}

En este ejemplo, en lugar de crear un nivel de jerarquía **%{::domain}** y una fuente de información para cada dominio, puedes obtener un resultado similar con una línea en la fuente de información **common**.

## Pasar variables a Hiera

Las variables de Hiera pueden venir de varias fuentes, dependiendo de cómo Hiera sea invocado.

### Desde Puppet

Cuando Hiera es usado con Puppet, recibe **automáticamente todas** las [variables](http://docs.puppetlabs.com/puppet/latest/reference/lang_variables.html) que Puppet tiene en ese momento. Esto incluye [facts y variables integradas](http://docs.puppetlabs.com/puppet/latest/reference/lang_variables.html#facts-and-built-in-variables), como también variables del scope actual. La mayoría de los usuarios interpolarán casi exclusivamente facts y variables integradas en su configuración y datos de Hiera.

+ Remueve el signo dólar **$** de Puppet cuando utilizas estas variables en Hiera. Es decir que una variable llamada **$::clientcert** en Puppet, en Hiera se llama **::clientcert**.
+ Se puede acceder a las variables de Puppet por su [nombre abreviado o su nombre completo](http://docs.puppetlabs.com/puppet/latest/reference/lang_variables.html#naming).

#### *Buenas prácticas*

+ Por lo general, evita hacer referencia a [**variables locales establecidas por el usuario**](http://docs.puppetlabs.com/puppet/latest/reference/lang_variables.html#facts-and-built-in-variables) desde Hiera. En lugar de eso utiliza **facts, variables integradas, variables top-scope, variables del scope del nodo**, o **variables del ENC** siempre que sea posible.
+ Cuando sea posible, haz referencia a las variables por su [**nombre completo**](http://docs.puppetlabs.com/puppet/latest/reference/lang_variables.html#naming) (ej.: **%{::environment}** y **%{::clientcert}**) para asegurarte que sus valores no estén enmascarados por scopes locales.

Estas dos pautas harán a Hiera más predecible, y pueden ayudar a que no se mezcle información con código en tus manifiestos Puppet.


### Desde la línea de comandos

Cuando llamas desde la línea de comandos, Hiera asume por defecto que no tiene variables disponibles. Puedes especificar variables individuales, o un archivo o servicio desde el cual obtener un “scope” completo de variables. Mira [usos de la línea  de comandos](http://docs.puppetlabs.com/hiera/1/command_line.html) para más detalles.

### Desde Ruby

Cuando llamas a Hiera desde código Ruby, puedes pasar un “scope” completo de variables como el tercer argumento para el método de **#lookup (búsqueda)**. La firma completa de **#lookup** es:

	hiera_object.lookup(key, default, scope, order_override=nil, resolution_type=:priority)
