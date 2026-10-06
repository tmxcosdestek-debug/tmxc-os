# TMXC OS - Guía de Instalación en Hardware

## ⚠️ Advertencia de Seguridad

**IMPORTANTE:** La instalación de TMXC OS en hardware físico implica modificar el firmware del dispositivo, desbloquear bootloaders y flashear imágenes de sistema operativo personalizadas. Estas acciones pueden:

- Anular la garantía del fabricante del dispositivo
- Posiblemente brickar el dispositivo si se realizan incorrectamente
- Exponer su dispositivo a riesgos de seguridad si no se toman las precauciones adecuadas
- Resultar en pérdida permanente de datos

**Proceda bajo su propio riesgo.** El equipo de desarrollo de TMXC OS no es responsable de ningún daño a su dispositivo o pérdida de datos. Siempre haga una copia de seguridad de sus datos antes de continuar.

## Requisitos Previos

### Requisitos de Hardware
- Dispositivo compatible con TMXC OS (ver [DEVICE_COMPATIBILITY.md](../../DEVICE_COMPATIBILITY.md))
- Cable de datos USB (para conexión a computadora)
- Computadora con conexión a internet
- Carga de batería mínima del 50% en el dispositivo objetivo

### Requisitos de Software
- Clave de activación de TMXC OS (obtener de tmxc.os.destek@gmail.com)
- Herramientas de plataforma (ADB/Fastboot) para su computadora
- Imagen de dispositivo TMXC OS (archivo .img o .bin)
- Herramienta de desbloqueo de bootloader específica del dispositivo (varía según el fabricante)

## Proceso de Instalación

### Paso 1: Obtener Clave de Activación

Antes de instalar TMXC OS, debe obtener una clave de activación válida:

1. Envíe un correo electrónico a **tmxc.os.destek@gmail.com**
2. Incluya el modelo y número de serie de su dispositivo
3. Espere la entrega de la clave de activación (típicamente 24-48 horas)
4. Mantenga su clave de activación segura - la necesitará durante el primer arranque

### Paso 2: Preparar su Computadora

**Windows:**
```powershell
# Descargar platform-tools del sitio web de Android Developer
# Extraer a C:\platform-tools
# Agregar al sistema PATH
```

**Linux:**
```bash
sudo apt-get install android-tools-adb android-tools-fastboot
```

**macOS:**
```bash
brew install android-platform-tools
```

### Paso 3: Habilitar Opciones de Desarrollador

1. Vaya a **Configuración** → **Acerca del Teléfono**
2. Toque **Número de compilación** 7 veces para habilitar Opciones de Desarrollador
3. Vuelva a **Configuración** → **Opciones de Desarrollador**
4. Habilite **Depuración USB**
5. Habilite **Desbloqueo OEM** (si está disponible)

### Paso 4: Desbloquear Bootloader

**⚠️ ADVERTENCIA:** Esto borrará todos los datos en su dispositivo.

#### Para la mayoría de dispositivos Android:

```bash
# Reiniciar a bootloader
adb reboot bootloader

# Desbloquear bootloader (comando varía según el fabricante)
fastboot oem unlock
# O
fastboot flashing unlock
```

#### Instrucciones Específicas del Dispositivo:

**Dispositivos Samsung:**
- Descargar herramienta Odin
- Descargar paquete de desbloqueo de bootloader Samsung
- Seguir proceso de desbloqueo específico del fabricante

**Dispositivos Xiaomi:**
- Solicitar permiso de desbloqueo del sitio web Mi Unlock
- Usar herramienta Mi Flash Unlock
- Seguir instrucciones en pantalla

**Google Pixel:**
- Habilitar desbloqueo OEM en Opciones de Desarrollador
- Usar `fastboot flashing unlock`

**OnePlus:**
- Habilitar Reinicio Avanzado en Opciones de Desarrollador
- Usar `fastboot oem unlock`

### Paso 5: Entrar en Modo Fastboot

```bash
# Desde estado apagado
# Mantener presionado Volumen Abajo + Botón de encendido simultáneamente
# O usar ADB
adb reboot bootloader
```

Verificar conexión:
```bash
fastboot devices
```

### Paso 6: Flashear Imagen TMXC OS

```bash
# Flashear la imagen TMXC OS en la partición apropiada
fastboot flash boot tmxc_os_boot.img
fastboot flash system tmxc_os_system.img
fastboot flash vendor tmxc_os_vendor.img

# O flashear imagen combinada
fastboot flash boot tmxc_os_combined.img
```

### Paso 7: Flashear Imagen de Recuperación (Opcional)

```bash
fastboot flash recovery tmxc_os_recovery.img
```

### Paso 8: Reiniciar al Sistema

```bash
fastboot reboot
```

### Paso 9: Configuración del Primer Arranque

1. **TMXC OS arrancará** (el primer arranque puede tomar 5-10 minutos)
2. **Selección de Idioma** - Elija su idioma preferido
3. **Configuración de Red** - Conecte a Wi-Fi o red móvil
4. **Activación** - Ingrese su clave de activación cuando se le solicite
5. **Configuración del Dispositivo** - Complete la configuración inicial

## Solución de Problemas

### El Dispositivo No Arranca

**Si el dispositivo está atascado en bootloop:**
```bash
# Reiniciar a recuperación
adb reboot recovery

# Borrar datos/restablecer de fábrica
# Reiniciar a bootloader
fastboot flash boot tmxc_os_boot.img
```

### Comandos Fastboot No Reconocidos

- Asegúrese de que platform-tools estén en su PATH
- Intente usar la ruta completa al ejecutable fastboot
- Verifique que el cable USB sea un cable de datos, no solo de carga

### Fallo al Desbloquear Bootloader

- Verifique que su dispositivo soporte desbloqueo de bootloader
- Verifique si aplican restricciones del operador
- Contacte al fabricante para el código de desbloqueo si es necesario

### Clave de Activación Inválida

- Verifique que ingresó la clave correctamente
- Contacte tmxc.os.destek@gmail.com si la clave parece inválida
- Asegúrese de que el modelo de su dispositivo coincida con el registrado

## Post-Instalación

### Verificar Instalación

```bash
# Verificar versión de TMXC OS
adb shell tmxc_version

# Verificar estado del sistema
adb shell tmxc_status
```

### Configuración de Seguridad

1. Habilitar autenticación biométrica
2. Configurar bóvedas de cifrado
3. Configurar modo fantasma si se desea
4. Configurar ajustes de firewall neural

### Actualizar Sistema

```bash
# Verificar actualizaciones
adb shell tmxc_update check

# Aplicar actualizaciones
adb shell tmxc_update install
```

## Recuperación y Restauración

### Restaurar Firmware Original

Si necesita restaurar su dispositivo al firmware de stock:

1. Descargue el firmware stock para su dispositivo
2. Reinicie a modo fastboot
3. Flashear imágenes stock:
```bash
fastboot flash boot stock_boot.img
fastboot flash system stock_system.img
fastboot flash vendor stock_vendor.img
fastboot flash recovery stock_recovery.img
```

### Recuperación de Emergencia

Si el dispositivo está brickeado:
- Use herramientas de unbrick específicas del dispositivo
- Contacte soporte de TMXC OS
- Considere servicio de reparación profesional

## Recursos Adicionales

- [Lista de Compatibilidad de Dispositivos](../../DEVICE_COMPATIBILITY.md)
- [Documentación de Seguridad](../security/activation_protocol.md)
- [Guía de Solución de Problemas](troubleshooting.md)
- [Foro de la Comunidad](https://community.tmx-os.org)

## Soporte

Para problemas de instalación:
- **Correo:** tmxc.os.destek@gmail.com
- **Documentación:** [docs.tmx-os.org](https://docs.tmx-os.org)
- **Comunidad:** [community.tmx-os.org](https://community.tmx-os.org)

---

**Última Actualización:** 2026-07-15  
**Versión de TMXC OS:** 1.0.0  
**Versión de la Guía:** 1.0
