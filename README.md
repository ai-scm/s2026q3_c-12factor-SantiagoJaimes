# The Twelve-Factor App: los 4 factores que elijo

**Autor:** Santiago Jaimes · **Semillero:** s2026q4c · **Referencia:** [12factor.net](https://12factor.net/es/)

Elegí estos cuatro factores porque son los que ya nos han dado problemas reales en los ejercicios del semillero: el HOL1 (Lambda, S3, Glue y Athena), OpenSearch con Docker Compose y el diseño C4 del bot de dailies para MS Teams. Ese bot probablemente lo tendremos que implementar después.

## Diagrama

<img width="1662" height="1020" alt="Assignment_12factor_SantiagoJaimes drawio" src="https://github.com/user-attachments/assets/cde1ac5a-336a-4462-8412-be4b346cc842" />



## III. Configuración: guardarla en el entorno

La configuración que cambia entre personas o entornos no debe estar escrita en el código. En el HOL1, cada uno tuvo que editar el código de la Lambda para poner el nombre de su carpeta, y los nombres de los buckets estaban fijos en el script. Si esos valores se leen de variables de entorno de la Lambda (`INPUT_BUCKET`, `OUTPUT_PREFIX`), el mismo código sirve para todo el equipo y para cualquier entorno. Lo mismo pasa con las credenciales `admin/admin` del `docker-compose.yml` de OpenSearch y con los tokens del bot de Teams: deben ir en un `.env` fuera del repositorio o en un gestor de secretos, nunca en GitHub.

## II. Dependencias: declararlas y aislarlas explícitamente

Una aplicación no debe suponer que algo "ya está instalado". En el HOL1, la Lambda falló con `No module named 'pandas'` porque la librería no existía en el entorno de ejecución, y hubo que agregarla como una *layer*. Declarar las dependencias con versión fija (`requirements.txt`, `package.json`, una imagen de Docker con tag concreto en lugar de `latest`) hace que cualquiera pueda clonar el repositorio y reproducir el resultado. Eso es justo lo que nos piden cuando dicen "documenta los pasos para poder replicarlo".

## VI. Procesos: ejecutarlos sin estado

Los procesos no deben guardar en memoria nada que necesiten después; el estado va a un servicio de respaldo (una base de datos o S3). Lambda ya obliga a esto: cada ejecución empieza de cero y puede durar como máximo 15 minutos. Es clave para el bot de dailies, porque detectar que alguien reportó el mismo tema tres días seguidos o que no reportó exige guardar el historial en una base de datos. Si el estado viviera en memoria, se perdería con cada ejecución o reinicio.

## X. Paridad entre desarrollo y producción

Lo que corre en local debe parecerse lo más posible a lo que corre en la nube, para que no aparezca el "en mi máquina sí funcionaba". Docker Compose nos permite levantar OpenSearch en local con la misma imagen y configuración que usaríamos después en un servicio gestionado. En AWS, usar los mismos servicios, región (`us-east-1`) y convenciones de nombres (prefijo `bld-tseed-workshop-`) en pruebas y en producción reduce las sorpresas al desplegar.
