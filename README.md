# 🎬 Jellyfin + ARR Stack: Tu propio servidor multimedia con Docker

[![GitHub](https://img.shields.io/badge/GitHub-Repositorio-blue)](https://github.com/JLalib/jellyfin-arr-docker)
[![Docker](https://img.shields.io/badge/Docker-Jellyfin-blue)](https://hub.docker.com/r/linuxserver/jellyfin)
[![License](https://img.shields.io/badge/Licencia-MIT-green)](https://github.com/JLalib/jellyfin-arr-docker/blob/main/LICENSE)

## 📋 Descripción general

Este repositorio contiene la configuración necesaria para desplegar un servidor multimedia completo usando Jellyfin y las aplicaciones *ARR (Sonarr, Radarr, etc.) con Docker Compose, siguiendo el tutorial de GenByte para crear tu propio "Netflix casero" y ahorrar en servicios de streaming.

Con esta solución podrás:
- Gestionar y transmitir tu colección de películas, series, música y más
- Automatizar la descarga y organización de contenido multimedia
- Acceder a tu contenido desde cualquier dispositivo
- Mantener el control total sobre tus datos y privacidad

## ✨ Características principales

- **Jellyfin**: Servidor multimedia completo para organizar y transmitir tu colección
- **Sonarr**: Automatización de descarga y organización de series de televisión
- **Radarr**: Automatización de descarga y organización de películas
- **qBittorrent**: Cliente de torrents ligero y poderoso
- **Gluetun**: Cliente VPN para mantener tu tráfico privado y seguro
- **Actualizaciones automáticas**: Con Watchtower para mantener tus contenedores actualizados
- **Interfaz web moderna**: Accesible desde cualquier navegador
- **Seguridad**: Configuración recomendada para privacidad y rendimiento

## 📋 Requisitos del sistema

- Docker y Docker Compose instalados
- Al menos 4 GB de RAM recomendados (2 GB mínimo)
- Espacio en disco suficiente para tu biblioteca multimedia (se recomienda SSD para mejor rendimiento)
- Puerto 8096 disponible para Jellyfin (HTTP)
- Puerto 8989 disponible para Sonar
- Puerto 7878 disponible para Radarr
- Puerto 8080 disponible para qBittorrent Web UI
- Puerto 8888 disponible para Gluetun (si se usa VPN)

## 🐳 Instalación

### Docker Compose (Método recomendado)

Crea un archivo `docker-compose.yml` con el siguiente contenido:

```yaml
version: "3.8"

services:
  # VPN para privacidad y seguridad
  vpn:
    image: qmcgaw/gluetun
    container_name: vpn
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=custom
      - OPENVPN_USER=tu_usuario_vpn
      - OPENVPN_PASSWORD=tu_contraseña_vpn
      - # Opcional: Servidores WireGuard si prefieres WireGuard conf, etc.
    ports:
      - 8888:8888/tcp  # HTTP proxy
      - 8888:8888/udp  # SOCKS5 proxy
    volumes:
      - ./vpn/data:/gluetun
    restart: unless-stopped

  # Cliente de torrents
  qbittorrent:
    image: linuxserver/qbittorrent
    container_name: qbittorrent
    depends_on:
      - vpn
    network_mode: "service:vpn"  # Trampa a través de la VPN
    volumes:
      - ./qbittorrent/config:/config
      - ./downloads:/downloads
    restart: unless-stopped

  # Gestor de series
  sonarr:
    image: linuxserver/sonarr
    container_name: sonarr
    depends_on:
      - vpn
    network_mode: "service:vpn"  # Trampa a través de la VPN
    volumes:
      - ./sonarr/config:/config
      - ./series:/series
      - ./downloads:/downloads
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    restart: unless-stopped

  # Gestor de películas
  radarr:
    image: linuxserver/radarr
    container_name: radarr
    depends_on:
      - vpn
    network_mode: "service:vpn"  # Trampa a través de la VPN
    volumes:
      - ./radarr/config:/config
      - ./movies:/movies
      - ./downloads:/downloads
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    restart: unless-stopped

  # Servidor multimedia
  jellyfin:
    image: linuxserver/jellyfin
    container_name: jellyfin
    depends_on:
      - vpn
    network_mode: "service:vpn"  # Trampa a través de la VPN
    volumes:
      - ./jellyfin/config:/config
      - ./media:/media
      - ./series:/series
      - ./movies:/movies
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    ports:
      - 8096:8096   # HTTP
      - 8920:8920   # HTTPS (opcional)
    restart: unless-stopped

  # Actualizaciones automáticas
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_LABEL_ENABLE=true
    restart: unless-stopped
```

### Variables de entorno

Copia el archivo `.env.example` a `.env` y ajusta los valores según tus necesidades:

```bash
cp .env.example .env
```

Luego, edita el archivo `.env` para configurar:
- Credenciales de tu servicio VPN (si usas una)
- Zonas horarias
- UID/GID para permisos de archivos
- Otras configuraciones específicas

### Iniciar los servicios

```bash
docker compose up -d
```

Esto iniciará todos los contenedores en segundo plano.

## 🚀 Primeros pasos

1. **Instala Docker y Docker Compose** en tu servidor si aún no lo has hecho
2. **Clona este repositorio** o copia el archivo `docker-compose.yml` a tu servidor
3. **Configura las variables de entorno** en el archivo `.env` (copiado desde `.env.example`)
4. **Ejecuta** `docker compose up -d` para iniciar todos los servicios
5. **Accede a las interfaces web**:
   - Jellyfin: `http://tu-servidor:8096`
   - Sonarr: `http://tu-servidor:8989` (a través de la VPN)
   - Radarr: `http://tu-servidor:7878` (a través de la VPN)
   - qBittorrent: `http://tu-servidor:8080` (a través de la VPN)

6. **Configura cada aplicación**:
   - En Sonarr y Radarr, configura el cliente de descarga para apuntar a qBittorrent a través de la VPN
   - Configura las carpetas de medios para que apunten a tusdirectorios de series y películas
   - En Jellyfin, agrega tus carpetas de medios (/series y /movies) a la biblioteca

## 💡 Casos de uso

- **Servidor de películas y series**: Crea tu propio Netflix con tu colección personal
- **Descarga automática**: Sonarr y Radarr buscarán y descargarán automáticamente nuevos episodios y películas
- **Transmisión en cualquier dispositivo**: Accede a tu contenido desde teléfonos, tablets, smart TVs y navegadores
- **Organización automática**: Los archivos se renombran y organizan correctamente al descargarse
- **Privacidad**: Todo el tráfico pasa por VPN para proteger tu identidad

## 🔒 Privacidad y seguridad

Esta configuración está diseñada con la privacidad en mente:
- Todo el tráfico de las aplicaciones *arr y qBittorrent pasa por una VPN
- No se expone directamente el cliente de torrents a internet
- Se recomienda usar una VPN de confianza que no guarde logs
- Las contraseñas y credenciales se manejan mediante variables de entorno

## 🛠️ Mantenimiento

### Ver registros
```bash
docker compose logs -f
```

### Actualizar a la última versión
```bash
docker compose pull
docker compose up -d
```

### Reiniciar servicios
```bash
docker compose restart
```

### Ver estado de salud
```bash
docker compose ps
```

### Limpiar y comenzar desde cero
```bash
docker compose down -v  # Elimina contenedores, redes y volúmenes
# Luego vuelve a levantar con: docker compose up -d
```

## 📝 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para más detalles.

---

> ✨ **Nota**: Este repositorio contiene la configuración Docker y documentación extraída y adaptada del tutorial de GenByte: [Cómo crear tu propio servidor multimedia con Jellyfin y ahorrar dinero](https://genbyte.blogspot.com/2024/11/como-crear-tu-poprio-servidor.html)