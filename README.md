# 🛡️ AdGuard Home - DNS Local & Bloqueador de Publicidad

Este repositorio contiene la configuración altamente optimizada para desplegar **AdGuard Home** en una Intel NUC (Ubuntu 26.04 LTS) utilizando Docker Compose. Proporciona resolución DNS de ultra baja latencia y filtrado de contenido a nivel de red para todo el hogar.

## 🌐 Estrategia de Red e Infraestructura

Este stack utiliza de forma deliberada el modo de red **`network_mode: host`**.

### ¿Por qué se diseñó así?
* **Cero Latencia (Overhead Eliminado):** Al amarrarse directamente a la interfaz física de la NUC, el tráfico DNS evita el puente virtual de Docker (`bridge`) y las reglas de traducción de direcciones (`NAT/IPTables`). Esto ahorra entre **0.5 y 2.5 ms** por consulta, garantizando respuestas instantáneas.
* **Visibilidad de Clientes:** Permite que AdGuard Home identifique las direcciones IP reales de los dispositivos de tu casa (`192.168.10.X`) en el panel de estadísticas, en lugar de ver que todo el tráfico proviene de la IP interna de Docker.
* **Simplicidad de Puertos:** No requiere mapear puertos uno a uno, ya que el contenedor toma posesión directa de los puertos necesarios en la máquina host.

---

## 🛠️ Requisitos Previos en el Servidor (Ubuntu)

Dado que AdGuard Home se adueña del puerto **53** de la NUC, es indispensable desactivar el servicio DNS nativo de Ubuntu (`systemd-resolved`) si está activo, para evitar conflictos de puerto:

```bash
# 1. Desactivar y detener el servicio nativo
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved

# 2. Editar el archivo de resolución si es necesario para apuntar a un DNS externo temporal
# (o configurar /etc/resolv.conf con "nameserver 1.1.1.1")
