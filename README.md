# 🛡️ AdGuard Home - DNS Local & Bloqueador de Publicidad

Este repositorio contiene la configuración altamente optimizada para desplegar **AdGuard Home** en **ZBook (Ubuntu Server - IP 192.168.10.25)** utilizando Docker Compose. Proporciona resolución DNS de ultra baja latencia y filtrado de contenido a nivel de red para todo el hogar.

---

## 🌐 Estrategia de Red e Infraestructura

Este stack utiliza de forma deliberada el modo de red **`network_mode: host`**.

### ¿Por qué se diseñó así?
* **Cero Latencia (Overhead Eliminado):** Al amarrarse directamente a la interfaz física de ZBook (`192.168.10.25`), el tráfico DNS evita el puente virtual de Docker (`bridge`) y las reglas de traducción de direcciones (`NAT/IPTables`). Esto ahorra entre **0.5 y 2.5 ms** por consulta, garantizando respuestas instantáneas.
* **Visibilidad de Clientes:** Permite que AdGuard Home identifique las direcciones IP reales de los dispositivos de la red local (`192.168.10.X`) en el panel de estadísticas, en lugar de ver que todo el tráfico proviene de la IP interna de Docker.
* **Simplicidad de Puertos:** No requiere mapear puertos uno a uno, ya que el contenedor toma posesión directa de los puertos necesarios en la máquina host.

---

## 🔐 Gestión de Secretos & 1Password

Las credenciales de administración del panel web de AdGuard Home se encuentran resguardadas y sincronizadas en la Bóveda **`IA`** de 1Password:

* **Ítem en 1Password:** `zbook_docker_adguard_admin` (UUID: `swagoif7szhshukdkbafcpzuuy`)
* **Campos Registrados:**
  * `username`: Contraseña de usuario administrador (`snadales`).
  * `password`: Contraseña en texto plano para acceso web (`https://adguard.yarken.cl` / `http://192.168.10.25:8088`).
  * `bcrypt_hash`: Hash encriptado bcrypt (`$2b$10$X8YEVVgTvF3YqaK7PGfxfeyO2SDN9ixa/AF40MopL8yYgHzHeOmGu`).
  * `deploy_key_private` & `deploy_key_public`: Copia de respaldo de las llaves SSH de deploy para GitHub.

> **Nota de Arquitectura:** AdGuard Home almacena internamente la contraseña procesada como un hash bcrypt dentro de `conf/AdGuardHome.yaml`. La versión oficial en texto plano para el inicio de sesión reside en 1Password.

---

## 🔑 Autenticación Git, Deploy Key & Firma SSH

Este repositorio utiliza una **Deploy Key SSH dedicada** con permisos de escritura acotada únicamente a este repositorio, y firmado obligatorio de commits vía SSH:

* **Repositorio Remoto:** `git@github.com-zbook-adguard:snadales/AdGuard.git`
* **Llave SSH en ZBook:** `/root/.ssh/id_ed25519_zbook-adguard` (Privada) / `/root/.ssh/id_ed25519_zbook-adguard.pub` (Pública)
* **Alias SSH Host:** `github.com-zbook-adguard` (configurado en `/root/.ssh/config`)
* **Firma de Commits:** `gpg.format=ssh` y `user.signingkey=/root/.ssh/id_ed25519_zbook-adguard.pub`

---

## 🛠️ Requisitos Previos en el Servidor (Ubuntu)

Dado que AdGuard Home se adueña del puerto **53** de ZBook, es indispensable desactivar el servicio DNS nativo de Ubuntu (`systemd-resolved`) si está activo:

```bash
# 1. Desactivar y detener el servicio nativo
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved

# 2. Verificar resolución DNS en ZBook
cat /etc/resolv.conf
```
