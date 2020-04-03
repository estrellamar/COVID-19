
Datos oficiales de COVID-19 en España
=====================================

<!-- 
Readme.md is generated from Readme.Rmd. 
Please edit that file 

Pendiente:

- Establecer fecha y último fichero pdf en cabecera YAML
  (actualizar texto con rmarkdown::metadata?).
  
- Añadir enlaces
  
- Combinar tablas MSCBS


-->
El objetivo principal de [este repositorio](https://github.com/rubenfcasal/COVID-19) es facilitar el acceso a los datos del COVID-19 en España a los que pueden estar interesados en analizarlos empleando R. Además se incluye una pequeña recopilación de enlaces a recursos que pueden ser de interés.

En [COVID-19-tablas.html](COVID-19-tablas.html) se muestran las tablas disponibles a fecha de ***2020-04-03***. Las tablas (con un procesado mínimo) están almacenadas en los archivos:

-   [edadsexo.RData](edadsexo.RData): Datos por edad y sexo (MSCBS)

-   [acumulados.RData](acumulados.RData): Evolución diaria de casos por CCAA (ISCIII)

-   [COVID-19.RData](COVID-19.RData): Datos por CCAA (MSCBS)

***Importante***: Desde el **2020-03-26** se pueden descargar los datos oficiales acumulados en la página web [Situación de COVID-19 en España](https://covid19.isciii.es) del [Instituto de Salud Carlos III (ISCIII)](https://www.isciii.es).
Archivo: [serie\_historica\_acumulados.csv](https://covid19.isciii.es/resources/serie_historica_acumulados.csv) (también disponible en este repositorio [aquí](serie_historica_acumulados.csv); el archivo [COVID-19-descarga.R](COVID-19-descarga.R) contiene el código necesario para descargar e importar estos datos a R).

De todos modos continuaré con este repositorio centrándome en los datos por edad y sexo:

Desde la `Actualizacion_53_COVID-19.pdf` (2020-03-23) los archivos contienen nuevas tablas con la distribución de casos hospitalizados, ingresados en UCI y fallecidos por grupos de edad y sexo. La [Actualizacion\_64\_COVID-19.pdf](Actualizacion_64_COVID-19.pdf) (2020-04-03) contiene la distribución de 70.674 casos notificados con información de edad y sexo (en los datos consolidados a las 21:00 horas del 2020-04-02).

***Importante***: El siguiente paso será tratar de conseguir los datos por provincias (puedes colaborar a través de GitHub o enviando un correo a <rubenfcasal@gmail.com>).

-   Galicia: <https://galiciancovid19.info>

-   Castilla y León: <https://analisis.datosabiertos.jcyl.es/pages/coronavirus/descarga-de-datasets#situacin-actual>

-   País Vasco: <https://opendata.euskadi.eus/catalogo/-/evolucion-del-coronavirus-covid-19-en-euskadi>

Fuentes de los datos
--------------------

En un primer momento, al buscar datos oficiales solo encontré esta web del *Ministerio de Sanidad, Consumo y Bienestar Social*:

<https://www.mscbs.gob.es/profesionales/saludPublica/ccayes/alertasActual/nCov-China/situacionActual.htm>

donde se puede descargar un pdf con la situación actual (actualizado a las 13:00 durante la semana y a las 12:00 el fin de semana; en la web también hay actualizaciones a otras horas).

Haciendo pruebas, vi que se podían descargar los documentos desde la actualización 31. El código y los documentos pdfs los podéis encontrar en este repositorio. El archivo `COVID-19-descarga.R` contiene el código necesario para descargar los pdfs.

Posteriormente, gracias a [este comentario](https://hypatia.math.ethz.ch/pipermail/r-help-es/2020-March/013753.html) en la lista de correo de [R-Hispano](http://r-es.org), descubrí otro repositorio que contiene los datos: <https://github.com/datadista/datasets/tree/master/COVID%2019> (de donde pude descargar el fichero `Actualizacion_44_COVID.pdf` que no encontré en la web oficial).

Otros enlaces que pueden ser de interés (ver Sección [Enlaces](#enlaces)):

-   [Instituto de Salud Carlos III (ISCIII)](https://www.isciii.es)

    -   [Situación de COVID-19 en España](https://covid19.isciii.es)

    -   [serie\_historica\_acumulados.csv](https://covid19.isciii.es/resources/serie_historica_acumulados.csv)

    -   [Informes COVID-19 del Centro Nacional de Epidemiología](https://www.isciii.es/QueHacemos/Servicios/VigilanciaSaludPublicaRENAVE/EnfermedadesTransmisibles/Paginas/InformesCOVID-19.aspx)

Preparación de los datos
------------------------

### Instalación de los paquetes necesarios

Para extraer las tablas desde R se emplea el paquete [tabulizer](https://docs.ropensci.org/tabulizer), que depende del paquete [rJava](https://rforge.net/rJava). Otro paquete que puede ser de utilidad es [pdftools](https://docs.ropensci.org/pdftools), empleado actualmente para extraer las fechas.

Estos dos últimos paquetes se pueden instalar desde CRAN, pero [rJava](https://cran.r-project.org/package=rJava) necesitaría tener instalado previamente el Java Runtime Environment correspondiente al equipo y a la versión de R (e.g [JRE de 64 bits para Windows](https://www.java.com/es/download/windows-64bit.jsp)).

Para instalar [tabulizer](https://docs.ropensci.org/tabulizer) se puede emplear el paquete [devtools](https://devtools.r-lib.org):

    devtools::install_github( c("ropenscilabs/tabulizerjars", "ropenscilabs/tabulizer"), 
                              INSTALL_opts = "--no-multiarch" )

Para instalar el resto de paquetes empleados (puede ser recomendable empezar por aquí) basta con ejecutar en la consola de R:

    pkgs <- c('rJava', 'pdftools', 'devtools', 'dplyr', 'DT')
    install.packages(setdiff(pkgs, installed.packages()[,"Package"]),
                     dependencies = TRUE)

    # Si aparecen errores (debidos a incompatibilidades con versiones anteriores de otros paquetes),
    # probar a ejecutar en lugar de lo anterior:
    # install.packages(pkgs, dependencies = TRUE) # Instala todos...

### Extracción

Las tablas por CCAA comienzan en `Actualizacion_35_COVID-19.pdf` (2020-03-03; en la tabla 3, que no se detecta). Las tablas por CCAA completas comienzan en `Actualizacion_36_COVID-19.pdf` (2020-03-04), aunque posteriormente hay cambios en los formatos de las tablas y de los archivos.

A partir de la [Actualizacion\_53\_COVID-19.pdf](Actualizacion_53_COVID-19.pdf) (2020-03-23) están disponibles nuevas tablas con la distribución de casos hospitalizados, ingresados en UCI y fallecidos por grupos de edad y sexo (a partir de los casos notificados que disponían de esa información).

El fichero `COVID-19-procesado.R` contiene el código necesario para extraer de los pdfs las tablas por grupo de edad y sexo (desde `Actualizacion_53_COVID-19.pdf`, 2020-03-23) y las tablas por CCAA (desde `Actualizacion_36_COVID-19.pdf`, 2020-03-04, hasta ¿hoy?).

### Tablas

Las tablas (con un procesado mínimo) están almacenadas en los archivos:

-   [edadsexo.RData](edadsexo.RData): Datos por edad y sexo (MSCBS)

-   [acumulados.RData](acumulados.RData): Evolución diaria de casos por CCAA (ISCIII)

-   [COVID-19.RData](COVID-19.RData): Datos por CCAA (MSCBS)

El fichero [COVID-19-tablas.html](COVID-19-tablas.html) contiene un listado (generado automáticamente a partir de [COVID-19-tablas.Rmd](COVID-19-tablas.Rmd)).

Colabora
--------

Work in progress... ***help needed!***: Especialmente en cuanto al modelado (actualmente está sesgado al campo de la estadística espacio-temporal, debido a la in/experiencia personal...)

El siguiente paso será tratar de conseguir datos por provincias y empezar a ajustar modelos (ver Sección [Enlaces](#enlaces))...

También puede ser de interés comparar los datos históricos del MSCBS con los del ISCIII (ver nota: "no se puede deducir que la diferencia entre un día y el anterior es el número de casos nuevos ya que esos casos pueden haber sido recuperados de fechas anteriores")...

Si quieres puedes ayudar a través de GitHub o enviando un correo a <rubenfcasal@gmail.com>.

Enlaces
-------

Work in progress... help needed!

-   [Acción Matemática contra el Coronavirus](http://matematicas.uclm.es/cemat/covid19)

-   Aplicación Shiny con estos datos: <https://covid19.citic.udc.es>

-   Datos CCAA:

    -   Galicia: <https://galiciancovid19.info>

    -   Castilla y León: <https://analisis.datosabiertos.jcyl.es/pages/coronavirus/descarga-de-datasets#situacin-actual>

    -   País Vasco: <https://opendata.euskadi.eus/catalogo/-/evolucion-del-coronavirus-covid-19-en-euskadi>

<br>

### COVID-19 y R

-   [Top 25 R resources on COVID-19 Coronavirus](https://www.statsandr.com/blog/top-r-resources-on-covid-19-coronavirus)

-   [Covid-19 interactive map (using R with shiny, leaflet and dplyr)](http://r-posts.com/covid-19-interactive-map-using-r-with-shiny-leaflet-and-dplyr)

-   [COVID-19 epidemiology with R](https://rviews.rstudio.com/2020/03/05/covid-19-epidemiology-with-r)

### Epidemiología (y áreas relacionadas) con R

-   <https://www.repidemicsconsortium.org>

-   [Model-based Geostatistics: Methods and Applications in Global Public Health (book)](https://www.crcpress.com/Model-based-Geostatistics-for-Global-Public-Health-Methods-and-Applications/Diggle-Giorgi/p/book/9781138732353) by P.J. Diggle and E. Giorgi (2019), [código R](https://sites.google.com/site/mbgglobalhealth/r-scripts)

-   [Spatio-Temporal Statistics with R (book)](https://spacetimewithr.org) by C.K. Wikle, A. Zammit-Mangion and N. Cressie (2019), [código R](https://spacetimewithr.org/code) (por si alguien se anima con modelos Bayesianos...)

-   [Forecasting: Principles and Practice (book)](https://otexts.com/fpp2), 2ª ed., by R.J. Hyndman and G. Athanasopoulos (2018).

-   [Epicalc\_Book](https://cran.r-project.org/doc/contrib/Epicalc_Book.pdf)

### Paquetes de R

Paquetes y otras herramientas...

-   [coronavirus](https://github.com/RamiKrispin/coronavirus)

-   [nCov2019](https://github.com/GuangchuangYu/nCov2019)

-   [forecast](https://pkg.robjhyndman.com/forecast)

Se puede realizar una búsqueda en <https://rseek.org>...
