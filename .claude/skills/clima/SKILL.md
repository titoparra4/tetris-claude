---
name: clima
description: Obtiene el clima actual usando wttr.in vía curl. Se activa cuando el usuario pide "el clima", "clima de X", "qué tiempo hace", "/clima" o pregunta por el pronóstico. Sin ubicación, usa Madrid, España por defecto; con ubicación, la usa tal cual.
---

# Clima

Obtiene el clima actual (o pronóstico corto) usando el servicio gratuito `wttr.in`, sin necesidad de API key.

## Uso

- Sin argumentos → usa Madrid, España por defecto.
- Con argumento → usa esa ciudad/ubicación (ej. `/clima Barcelona`, `/clima Ciudad de Mexico`).

## Instrucciones

1. Construir la URL:
   - Sin ubicación: `https://wttr.in/Madrid?format=j1`
   - Con ubicación: `https://wttr.in/<ciudad-url-encoded>?format=j1` (reemplazar espacios por `+`)
2. Ejecutar con curl (timeout corto, ya que es un servicio externo):
   ```bash
   curl -s --max-time 10 "https://wttr.in/Madrid?format=j1"
   ```
3. Del JSON devuelto, extraer y mostrar al usuario en español, de forma breve:
   - `current_condition[0].temp_C` (temperatura actual en °C)
   - `current_condition[0].FeelsLikeC` (sensación térmica)
   - `current_condition[0].weatherDesc[0].value` (descripción del clima)
   - `current_condition[0].humidity` (humedad %)
   - `current_condition[0].windspeedKmph` (viento en km/h)
   - `nearest_area[0].areaName[0].value` y `nearest_area[0].country[0].value` (para confirmar la ubicación detectada/usada)
4. Si `curl` falla o el servicio no responde (sin red, timeout, error HTTP), informar al usuario claramente que no se pudo obtener el clima y sugerir reintentar o especificar una ciudad.
5. No inventar datos climáticos si la petición falla.

## Ejemplo de salida esperada

```
Clima en Madrid, España:
- Condición: Parcialmente nublado
- Temperatura: 24°C (sensación 26°C)
- Humedad: 55%
- Viento: 12 km/h
```
