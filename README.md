# 🛡️ AdGuard Home - DNS Local & Bloqueador de Publicidad

Este repositorio contiene la infraestructura y configuración estandarizada para desplegar **AdGuard Home** en **ZBook (Ubuntu Server - IP 192.168.10.25)** utilizando Docker Compose. Proporciona resolución DNS de ultra baja latencia y filtrado de contenido a nivel de red para el hogar.

---

## 🌐 Estrategia de Red e Infraestructura

Este stack utiliza el modo de red **`network_mode: host`**.

### Razón de Diseño
* **Cero Latencia (Overhead Eliminado):** Al amarrarse directamente a la interfaz física de ZBook (`192.168.10.25`), el tráfico DNS evita el puente virtual de Docker (`bridge`) y las reglas de traducción de direcciones (`NAT/IPTables`), ahorrando entre **0.5 y 2.5 ms** por consulta.
* **Visibilidad de Clientes:** Permite que AdGuard Home identifique las direcciones IP reales de los dispositivos de la red local (`192.168.10.X`) en los logs y estadísticas.
* **Simplicidad de Puertos:** Toma posesión directa de los puertos necesarios en la máquina host.

---

## 🔐 Gestión de Secretos & Integración con 1Password

Ninguna contraseña ni credencial sensible se almacena en texto plano dentro de este repositorio o sus archivos de configuración.

* **Ítem en 1Password:** `zbook_docker_adguard_admin` (Bóveda: `IA`, UUID: `swagoif7szhshukdkbafcpzuuy`)
* **Referencias 1Password (`op://`):**
  * **Usuario Admin:** `op://IA/zbook_docker_adguard_admin/username`
  * **Contraseña Admin (Texto Plano):** `op://IA/zbook_docker_adguard_admin/password`
  * **Hash Encriptado (Bcrypt):** `op://IA/zbook_docker_adguard_admin/bcrypt_hash`
  * **Clave Pública Deploy SSH:** `op://IA/zbook_docker_adguard_admin/deploy_key_public`
  * **Clave Privada Deploy SSH:** `op://IA/zbook_docker_adguard_admin/deploy_key_private`
  * **Repositorio GitHub:** `op://IA/zbook_docker_adguard_admin/github_repository`

### Extracción de Secretos vía CLI / 1Password Connect API
```bash
# Lectura de usuario admin
op read op://IA/zbook_docker_adguard_admin/username

# Lectura de contraseña web
op read op://IA/zbook_docker_adguard_admin/password

# Consulta HTTP vía 1Password Connect Server API en ZBook
curl -s -H Authorization: Bearer $OP_CONNECT_TOKEN http://localhost:8080/v1/vaults/IA/items/swagoif7szhshukdkbafcpzuuy
```

> **Nota de Arquitectura:** AdGuard Home guarda nativamente la contraseña del usuario administrador procesada como un hash bcrypt en `conf/AdGuardHome.yaml`. La contraseña en texto plano para inicio de sesión vive únicamente en 1Password bajo la referencia `op://IA/zbook_docker_adguard_admin/password`.

---

## 🔑 Autenticación Git, Deploy Keys & Firma de Commits

Este repositorio está conectado de forma aislada a GitHub mediante una **Deploy Key SSH dedicada** y firma de commits obligatoria.

### Repositorio y Enlaces
* **GitHub Web:** [https://github.com/snadales/AdGuard](https://github.com/snadales/AdGuard)
* **Remoto SSH:** `git@github.com-zbook-adguard:snadales/AdGuard.git`

### Uso y Configuración de Llaves Deploy SSH
* **Ubicación en ZBook:** `/root/.ssh/id_ed25519_zbook-adguard` (Privada) / `/root/.ssh/id_ed25519_zbook-adguard.pub` (Pública)
* **Clave Pública SSH:**
  ```text
  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDqIzxodoF1CDBwPQpTvs3KvyIzN501DvnYV0xJI2Ghx zbook-adguard-deploy
  ```
* **Configuración Host SSH (`/root/.ssh/config`):**
  ```text
  Host github.com-zbook-adguard
      HostName github.com
      User git
      IdentityFile /root/.ssh/id_ed25519_zbook-adguard
      IdentitiesOnly yes
  ```
* **Firma SSH de Commits Git:**
  Todos los commits de este repositorio están autenticados y firmados mediante la llave SSH:
  ```bash
  git config user.name Sebastián Nadales
  git config user.email snadales@gmail.com
  git config gpg.format ssh
  git config user.signingkey /root/.ssh/id_ed25519_zbook-adguard.pub
  git config commit.gpgsign true
  ```

---

## 🛠️ Requisitos Previos en el Servidor (Ubuntu)

Dado que AdGuard Home utiliza el puerto **53** de ZBook, es indispensable desactivar el servicio DNS nativo de Ubuntu (`systemd-resolved`):

```bash
# 1. Desactivar y detener el servicio nativo
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved

# 2. Verificar resolución DNS en ZBook
cat /etc/resolv.conf
```
