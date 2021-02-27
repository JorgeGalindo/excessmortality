# Título del Proyecto

## Comenzando 🚀

Una plataforma de análisis de exceso de mortalidad en 2020 con datos de otros tipos; particularmente cambios en el PIB registrados por el FMI. El funcionamiento es sencillo: se extrae el dato consolidado de exceso de mortalidad en 2020 vs. 2015-19 siguiendo orden de prioridad de tres fuentes distintas; se aúna en una sola base de datos; y se procede a incorporar los datos económicos (descargados manualmente del FMI; no proporciona link usable) para combinarlos en una sola.

Para asegurar que no haya desajustes, existe un .csv "llave" que "traduce" los nombres de países entre sí (todo en inglés); para después sumarle otro que los mantiene en español.


### Pre-requisitos 📋

Los paquetes clásicos que verás en el script: tidyverse, ggplot2; además, para visualización estilo EsadeEcPol: ecpolstyle.


### Componentes ⚙️

input <- Aquí van los datos necesarios que no pueden ser adquiridos por link. También hay un csv que sirve de llave para alinear países por su nombre (countrykey.csv); junto a su Numbers en el que es más cómodo hacer actualizaciones conforme se añadan países (no hay otra solución). Se incluye asimismo el .xls y el .csv de los datos del FMI, que no disponen de link.

extraction-analysis.R <- Script para absorber y leer datos

analysis.Rmd <- Libreta para analizar

output <- csv de salida; Numbers para graficar a mano si se desea


## Autores ✒️

**Jorge Galindo** - *Responsable del repo*

También puedes mirar la lista de todos los [contribuyentes](https://github.com/excessmortality/project/contributors) quíenes han participado en este proyecto. 

## Licencia 📄

Licencia libre, úsalo como consideres :)
