## Traefik v3 Proxy Reverso con Docker

Despliegue y configuración de **Traefik v3** como proxy reverso y gestor de rutas automatizado mediante la API de Docker para tu infraestructura local (*self-hosted*).

---

## 📋 Precondiciones

Antes de desplegar este contenedor, asegúrate de cumplir con los siguientes requisitos en tu host Docker:

1. **Docker y Docker Compose** instalados en el sistema.
2. **Red externa de Traefik creada**: Este `docker-compose.yml` utiliza una red externa llamada `traefik` a la cual se conectarán el resto de servicios que desees exponer.
   
   Puedes crearla ejecutando:
   ```bash
   docker network create traefik
   ```
3. **Puertos libres en el Host**: Los puertos `80` (tráfico web HTTP) y `8080` (panel de control / dashboard) deben estar libres y disponibles en tu servidor.

---

## 📂 Volúmenes y Persistencia

El contenedor requiere acceso directo al socket de Docker para escuchar los eventos de despliegue y detectar automáticamente las etiquetas (*labels*) de los demás servicios:

* `/var/run/docker.sock`: Permite a Traefik interactuar con el demonio de Docker en tiempo real. 
  > ⚠️ **Nota de seguridad**: Dar acceso al socket de Docker implica que el contenedor tiene privilegios elevados sobre el host. Mantén la imagen y el sistema actualizados.

---

## 🔒 Consideraciones de Seguridad

Al desplegar Traefik con acceso al socket de Docker y paneles de control, es vital tener en cuenta las siguientes medidas de seguridad:

1. **Panel de Control Inseguro (`--api.insecure=true`)**: 
   - Esta configuración habilita el dashboard de Traefik y la API en el puerto `8080` **sin autenticación** y en texto plano (HTTP).
   - **Recomendación crítica**: Esta configuración está pensada principalmente para entornos de pruebas locales o redes de confianza (*home labs*). En entornos expuestos a internet, se debe deshabilitar el modo inseguro y configurar un router protegido con autenticación (middleware BasicAuth) y HTTPS.
2. **Exposición por Defecto (`exposedbydefault`)**:
   - En este compose, la línea `--providers.docker.exposedbydefault=false` está comentada. Esto significa que **cualquier contenedor** que levantes en tu infraestructura que esté conectado a la red `traefik` será expuesto automáticamente si no se indica lo contrario. 
   - **Buena práctica**: Se recomienda descomentar esa línea para que un contenedor solo sea accesible si explícitamente se le añade la etiqueta `traefik.enable=true`.
3. **Control de Puertos**: Traefik centraliza la entrada de todo el tráfico web abierto al exterior a través del puerto `80`, permitiendo que el resto de tus contenedores internos permanezcan completamente ocultos (sin puertos expuestos en el host).

---

## ⚙️ Configuración y Funcionamiento de Comandos (*Command*)

Los argumentos definidos en el bloque `command` configuran el comportamiento nativo de Traefik v3:

* `--api.insecure=true`: Activa la API de diagnóstico y el panel web de control.
* `--api.dashboard=true`: Habilita la interfaz gráfica web de Traefik (accesible en el puerto `8080`).
* `--providers.docker=true`: Indica a Traefik que utilice el proveedor de Docker para descubrir servicios mediante las etiquetas de los contenedores.
* `--entrypoints.web.address=:80`: Define el punto de entrada principal llamado `web` escuchando en el puerto `80` para el tráfico HTTP.

---

## 🚀 Puesta en Marcha

1. Crea la red externa obligatoria en tu servidor:
   ```bash
   docker network create traefik
   ```
2. Coloca este `docker-compose.yml` en el directorio de tu elección.
3. Despliega el proxy reverso ejecutando:
   ```bash
   docker compose up -d
   ```
4. Accede al panel de control de Traefik en tu navegador a través de: `http://<IP-de-tu-servidor>:8080` para comprobar qué contenedores están siendo descubiertos y enrutados.
