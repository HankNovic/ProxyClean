# ProxyClean

<!-- hy-mt2-i18n:start -->
[中文](./README.md) | [English](./README_en.md) | [日本語](./README_ja.md) | **Español**
<!-- hy-mt2-i18n:end -->


Almacén de datos de proxies SOCK5 recopilado públicamente y actualizado de forma continua.

## Resumen de datos

- Archivo del pool de proxies: `SOCKS5.txt`
- Archivo de datos brutos: `SOCKS5_RAW.txt`
- Enlace directo original (pool disponible): https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5.txt
- Enlace directo original (pool bruto): https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5_RAW.txt
- Verificación de conectividad: handshake SOCKS5 + prueba de destino SOCKS5 CONNECT
- Destinos predeterminados para la prueba CONNECT: `1.1.1.1:53`, `8.8.8.8:53`
- Nodo de prueba: **Xiamen, Fujian, China – Telecom**

Notas:
- Los proxies de la lista provienen de fuentes públicas y gratuitas.
- Este repositorio se encarga únicamente de la agregación, limpieza, mantenimiento activo y publicación, sin ofrecer ningún servicio de pago.

## Características del pool de proxies

- **Prioridad a proxies disponibles en China**: Se verifica la conectividad en el entorno de red de la China continental; solo aquellos que aprueben esta verificación pasan a formar parte del pool disponible.
- **Limpieza mediante agregación de múltiples fuentes**: Se combinan varias fuentes públicas de SOCKS5 para realizar una interpretación unificada, eliminación de duplicados y filtrado.
- **Clasificación por verificación concurrente**: Se ejecutan de forma simultánea las pruebas de conectividad y de latencia, mostrando como resultado los nodos que están disponibles recientemente y presentan una mayor estabilidad en la latencia.
- **Verificación real de disponibilidad**: Tras superar la prueba de handshake, se añade una prueba con destino SOCKS5 CONNECT; solo cuando tanto la prueba de handshake como la de CONNECT tienen éxito se considera que el proxy es válido.
- **Estrategia unificada de manejo de errores**: Se utiliza la misma clasificación de errores (blandos y duros) y se realizan operaciones de eliminación de duplicados tanto para la prueba de handshake como para la de CONNECT, lo que reduce los falsos positivos causados por fluctuaciones temporales en la red.
- **Actualizaciones continuas de mantenimiento activo**: Se realizan comprobaciones periódicas de mantenimiento activo entre las extracciones completas, eliminando automáticamente los nodos que ya no son funcionales.
- **Mantenimiento dinámico**: El archivo `SOCKS5.txt` se actualiza de forma continua para reflejar, en la medida de lo posible, el estado de disponibilidad más reciente.

## Fuentes de los proxies (Agradecimientos)

Actualmente se agrupan principalmente las siguientes fuentes públicas de SOCKS5 (sin orden de prioridad):

- [proxifly/free-proxy-list](https://github.com/proxifly/free-proxy-list)  
- [TheSpeedX/PROXY-List](https://github.com/TheSpeedX/PROXY-List)  
- [roosterkid/openproxylist](https://github.com/roosterkid/openproxylist)  
- [hookzof/socks5_list](https://github.com/hookzof/socks5_list)  
- [gfpcom/free-proxy-list](https://github.com/gfpcom/free-proxy-list)  
- [dpangestuw/Free-Proxy](https://github.com/dpangestuw/Free-Proxy)

> Nota: Este repositorio no ofrece garantía de calidad para ninguna fuente en particular, sino que realiza una selección técnica únicamente basada en los resultados reales de las pruebas.

## Formato de los archivos

Tanto `SOCKS5.txt` como `SOCKS5_RAW.txt` utilizan formato de texto plano:

IP:puerto

- Cada línea corresponde a un proxy SOCKS5.
- Se utiliza dos puntos y coma `:` en inglés para separar la IP de el puerto.
- De forma predeterminada, no se incluyen nombres de usuario/contraseñas.
- Los clientes suelen utilizar los protocolos `socks5` o `socks5h`.

## Modo de uso

### 1) Leer los proxies

Leer línea por línea desde `SOCKS5_RAW.txt`, por ejemplo:

1.2.3.4:1080

### 2) Prueba desde la línea de comandos (curl)

```bash
curl --socks5 1.2.3.4:1080 https://api.ipify.org
```

Si desea que la resolución de dominios también se realice a través del proxy:

```bash
curl --socks5-hostname 1.2.3.4:1080 https://api.ipify.org
```

### 3) Método mediante variables de entorno en Linux

```bash
export ALL_PROXY="socks5://1.2.3.4:1080"
export http_proxy="socks5://1.2.3.4:1080"
export https_proxy="socks5://1.2.3.4:1080"

curl https://api.ipify.org
```

### 4) Configuración de proxy individual para Git

```bash
git config --global http.proxy "socks5://1.2.3.4:1080"
git config --global https.proxy "socks5://1.2.3.4:1080"
```

Desconfigurar:

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 5) Python (requests)

```python
import requests

proxy = "1.2.3.4:1080"
proxies = {
    "http": f"socks5://{proxy}",
    "https": f"socks5://{proxy}",
}

resp = requests.get("https://api.ipify.org", proxies=proxies, timeout=10)
print(resp.text)
```

> Antes de utilizar la capacidad de proxy SOCKS de `requests`, instale `requests[socks]` o `pysocks`.

### 6) Clientes comunes

- **Clash / Clash.Meta**: Agregue un nodo de tipo `type: socks5` e indique el `server` y el `port`.  
- **v2rayN / Qv2ray**: Añada un nodo Socks o una reenvío local que apunte a `ip:port`.  
- **Complementos de navegador como SwitchyOmega**: Cree un nuevo modo escenario SOCKS5 e introduzca la IP y el puerto.

Se recomienda incorporar mecanismos de intentos repetidos en caso de fallo, polling y verificación de estado en el uso práctico.

## Actualizaciones y disponibilidad

- Extracción y limpieza completa: se realiza aproximadamente una vez por hora por defecto.
- Verificación periódica de actividad: se lleva a cabo una validación de disponibilidad de forma cíclica entre dos extracciones completas.
- Guardián de red: interruptor de detección en línea; si hay anomalías en la detección de red, se suspende la verificación actual y se reanuda automáticamente al normalizarse la conexión.
- Limpieza de historial: se eliminan los datos vencidos según el número de días de retención (30/15/7/1), aplicando las políticas correspondientes a las estadísticas históricas y los registros de operación.
- Notas sobre disponibilidad: `SOCKS5.txt` solo representa los resultados de detección actuales o recientes, sin garantía de estabilidad a largo plazo.

Escenarios de uso: extracción temporal, entornos de prueba, aprendizaje e investigación.  
No se recomienda su uso directo en entornos de producción con altos requisitos de conformidad y estabilidad.

## Declaración de riesgos y instrucciones de uso

- Los proxies provienen de la red pública, por lo que no se garantiza su autenticidad, conformidad ni seguridad.  
- El uso de proxies puede conllevar riesgos como **fuga de privacidad, controles de seguridad en las cuentas y filtración de datos**; por favor, realice su propia evaluación.  
- Respete estrictamente las leyes y regulaciones de su país, así como los términos de servicio de la plataforma objetivo, y no los utilice para fines ilegales o irregulares.  
- Se recomienda usarlos únicamente para el aprendizaje de protocolos, depuración de redes y pruebas de investigación.
