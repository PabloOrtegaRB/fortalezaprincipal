# fortalezaprincipal — Pipeline de contenido crypto (video + análisis)

Repo de trabajo para el pipeline de "nacimiento de un video": análisis de mercado →
idea/generación → edición → publicación. Arranca en la rama `pipeline-setup`.

## Etapas y herramientas

| Etapa | Herramienta | Dónde vive |
|---|---|---|
| 1. Análisis de mercado | Binance MCP (klines, tickers, cuentas) | Conectado directo en Claude Code, no vendoreado aquí |
| 1b. Análisis técnico | Skills `skill-01` a `skill-11` (estructura, EMA, RSI, MACD, volumen, divergencias, liquidez, riesgo, multi-timeframe, sesgos) | Plugin `anthropic-skills` ya disponible en Claude Code — no son archivos locales, se invocan por nombre |
| 2. Idea + generación | Higgsfield MCP (`generate_video`, `generate_audio`, workflow `faceless-video`) | Conectado directo en Claude Code |
| 3. Edición | Agentes `ffmpeg-clip-team` (este repo, `.claude/agents/ffmpeg-clip-team/`) | Vendoreados desde `davila7/claude-code-templates` |
| 4. Publicación | Metricool MCP (scheduling, analytics) | Conectado directo en Claude Code |

## Contenido de este repo

- `.claude/agents/ffmpeg-clip-team/` — agentes de edición: `video-editor`, `social-media-clip-creator`
  (recorte 9:16 para reel + subtítulos + thumbnail), `audio-mixer`, `audio-quality-controller`,
  y agentes de podcast (`podcast-transcriber`, `podcast-metadata-specialist`, `podcast-content-analyzer`,
  `timestamp-precision-specialist`) por si algún contenido es formato podcast.
- `.claude/mcps/alphai.json` — MCP de noticias financieras pre-analizadas (contexto adicional
  para la Etapa 1, complementa a Binance).

## Regla de sesgo EMA (Etapa 1b)

Definida por el usuario: cruce de EMA10/20 sobre EMA50 activa **long** solo si el precio está
por encima de EMA200; cruce por debajo activa **short** solo si el precio está por debajo de
EMA200. Fuera de esas condiciones: sin sesgo claro, no se sugiere entrada.

## Resultado final del proceso

Dos archivos por cada pieza de contenido:
- `youtube_full.mp4` — video completo, 16:9, ~1-3 min (según duración elegida en `faceless-video`)
- `reel_short.mp4` — short/reel, 9:16, <60s, subtitulado, recortado del mismo material con
  `social-media-clip-creator`

## Equipo de trading (`.claude/agents/trading/`) — solo análisis y alertas

| Rol pedido por el usuario | Cómo quedó cubierto |
|---|---|
| RSI + avisar en divergencia | `skill-03-rsi` + `skill-06-divergence` (ya existían, plugin `anthropic-skills`) |
| Cruce EMA 10/20/50/200 (long/short bias) | **`ema-trend-analyst`** (nuevo, este repo) |
| Volumen | `skill-05-volume` (ya existía) |
| Control de entradas / gestión de riesgo | `skill-08-risk` (ya existía, tiene poder de veto) |
| Métricas de todo el listado de futuros USDT | **`usdt-futures-screener`** (nuevo, este repo) |
| Generar la entrada (veredicto compuesto) | **`signal-orchestrator`** (nuevo, este repo) — combina todo, nunca ejecuta |
| Auditor de sesgos | `skill-11-market-psychology` (ya existía) |
| Vigilar la posición cada cierto tiempo | **`position-watch`** (nuevo, este repo) — solo alerta, nunca cierra la posición |
| Ejecutar la entrada | **No existe y no se va a construir** — colocar órdenes reales queda fuera de mis límites sin importar el contexto o la autorización dada |
| Salida automática / auto take-profit | **No existe** — `position-watch` es el sustituto: avisa, nunca cierra la posición solo |

## Pendiente / no incluido todavía

- Video real de ejemplo: bloqueado por falta de créditos de Higgsfield (10 disponibles,
  se necesitan 65 para un clip de prueba de 10s) — pendiente de que el usuario recargue
  créditos o autorice un modelo/duración más barata.
- **Agente iniciador/cron**: dispara el proceso completo solo, sin presencia del usuario,
  cada cierto tiempo. Se agrega recién cuando el video de ejemplo salga como se espera y el
  resto de los agentes funcionen bien en la marcha blanca — no antes.
