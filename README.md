# SonarRescate v0.4

**Herramienta acústica de búsqueda y rescate en estructuras colapsadas**

Aplicación web progresiva (PWA) diseñada para primeros respondedores en zonas de desastre sísmico donde no se dispone de equipos USAR profesionales en las primeras 48 horas críticas.

---

## Funcionalidades

- **Escucha amplificada** — ganancia configurable ×10–×100, calibración automática de línea base de ruido ambiental
- **Detección de señales débiles** — captura golpes, chasquidos, respiración forzada y voz amortiguada
- **Estimador de distancia** — modelo acústico en concreto armado (~3 dB/cm a 1.5 kHz), 5 zonas de 0–5+ metros
- **Protocolo INSARAG** — 3 pulsos de señal + silencio de escucha con visualización de ciclo
- **GPS + Mapa de calor** — georreferencia cada evento, visualización en OpenStreetMap
- **Exportación** — GeoJSON (QGIS/ArcGIS), CSV (Excel), TXT (coordinación de campo)
- **Funciona offline** — Service Worker cachea todo después de la primera carga
- **Instalable** — PWA compatible con Android e iOS sin tienda de aplicaciones

---

## Uso rápido

### Opción 1 — Usar directamente en el navegador

Abre `index.html` en cualquier navegador moderno. No requiere servidor.

### Opción 2 — Desplegar en GitHub Pages (gratis, acceso desde cualquier dispositivo)

```bash
# 1. Fork o clona este repositorio
git clone https://github.com/TU_USUARIO/sonar-rescate.git

# 2. Ve a Settings → Pages → Source: main / root
# 3. La app queda disponible en:
#    https://TU_USUARIO.github.io/sonar-rescate/
```

### Opción 3 — Netlify / Vercel (1 clic)

Arrastra la carpeta del proyecto a netlify.com/drop — queda publicada en segundos con HTTPS.

### Opción 4 — Servidor local

```bash
# Python 3
python3 -m http.server 8080

# Node.js
npx serve .
```

---

## Instalar como app en el dispositivo

**Android (Chrome):**
Menú del navegador (⋮) → "Añadir a pantalla de inicio" → Instalar

**iPhone / iPad (Safari):**
Botón Compartir (□↑) → "Añadir a pantalla de inicio"

---

## Estructura del proyecto

```
sonar-rescate/
├── index.html          # App completa (HTML + CSS + JS en un solo archivo)
├── manifest.json       # Configuración PWA
├── sw.js               # Service Worker para offline
├── icons/
│   ├── icon-192.png    # Ícono app Android
│   └── icon-512.png    # Ícono app splash screen
└── README.md           # Este archivo
```

---

## Arquitectura técnica

### Cadena de audio
```
Micrófono → GainNode (×60) → BiquadFilter HPF 150Hz → AnalyserNode → DSP
```

### Algoritmo de detección
- **Peak detection** (70%) + **RMS** (30%) sobre ventana de 256 samples
- **Calibración automática** de ruido base (percentil 70 de 3 segundos de muestreo)
- **Umbral dinámico**: max(umbral_manual, baseline + 8 dB)
- **Cooldown** de 1.4 segundos entre eventos para evitar duplicados

### Modelo de distancia acústica
Basado en atenuación en concreto armado (~3 dB/cm a 1.5 kHz):

| Nivel detectado | Distancia estimada | Contexto |
|---|---|---|
| > -20 dB | < 50 cm | Contacto directo |
| -20 a -32 dB | 0.5–1.5 m | Pared o losa delgada |
| -32 a -50 dB | 1.5–3 m | 1–2 capas de escombro |
| -50 a -65 dB | 3–5 m | Múltiples capas |
| < -65 dB | > 5 m | Incierto / ruido |

> **Nota:** Los valores son estimaciones orientativas para golpes activos.
> La respiración pasiva reduce el alcance efectivo en ~60%.
> Esta herramienta no reemplaza equipos USAR profesionales (Delsar, LifeLocator).

### Cadena GPS → Mapa
```
watchPosition() → posActual → evento detectado → Feature GeoJSON → 
marcador Leaflet + blob mapa de calor Canvas
```

---

## Compatibilidad

| Plataforma | Soporte | Notas |
|---|---|---|
| Android Chrome | ✅ Completo | Instalable como PWA |
| Android Firefox | ✅ Audio + GPS | Instalación manual |
| iPhone Safari | ✅ Completo | Añadir a pantalla de inicio |
| Chrome Desktop | ✅ Completo | Instalable |
| Firefox Desktop | ✅ Completo | Sin instalación PWA |

**Requerimientos mínimos:** Navegador con Web Audio API + getUserMedia + Geolocation API

---

## Contribuir

Este proyecto es de código abierto bajo licencia MIT. Las contribuciones son bienvenidas:

1. **Issues** — reporta errores o propone mejoras
2. **Pull requests** — sigue las convenciones del código existente
3. **Testing en campo** — los datos de validación con bomberos o defensa civil son valiosos

### Roadmap

- [ ] Análisis espectral FFT con visualización de frecuencias
- [ ] Triangulación de posición con múltiples puntos de medición
- [ ] Modo coordinador — exportar mapa en tiempo real a dispositivo central
- [ ] Soporte para micrófono externo de contacto (geófono piezoeléctrico via USB-C)
- [ ] Internacionalización (EN, PT, FR)
- [ ] Integración con protocolo ICS-213 (formularios USAR)
- [ ] Versión nativa Android (Capacitor / Kotlin)

---

## Licencia

MIT License — libre uso, modificación y distribución con atribución.

---

## Créditos y referencias técnicas

- Protocolo de búsqueda: INSARAG Guidelines 2020, Volumen II
- Atenuación acústica en concreto: ISO 10140-2:2021
- Web Audio API: W3C Specification
- Mapas: © OpenStreetMap contributors
- Leaflet.js: BSD 2-Clause License

---

*Desarrollado para contextos de desastre en América Latina donde los equipos USAR profesionales tardan 24–72 horas en llegar y la comunidad debe actuar en las primeras horas críticas.*
