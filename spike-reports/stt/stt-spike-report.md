# Evaluación de Speech-to-Text (STT) para carga de transacciones por voz

## Objetivo

Investigar y seleccionar una solución de **Speech-to-Text** para incorporar a TecaTrack,
priorizando alternativas **open source** o basadas en **modelos abiertos**, con preferencia
por opciones consumibles como **servicio gestionado (API)** para no sobrecargar la memoria
RAM de los equipos locales.

El caso de uso es la **carga de transacciones por voz**: el usuario dicta una operación
(monto, banco origen/destino, concepto) y el sistema la transcribe para precargar un
borrador de transacción. La grabación se hará desde el navegador (JS / `MediaRecorder`) y
el procesamiento es **asíncrono** respecto a la respuesta del backend.

## Alcance

- Investigar alternativas open source.
- Evaluar opciones como servicio (API) y alternativas self-hosted.
- Comparar precisión de transcripción para el español.
- Revisar facilidad de integración con la arquitectura actual.
- Realizar una prueba de concepto con la alternativa elegida.

## Restricciones del proyecto

Estas restricciones condicionan la decisión y descartan parte de las alternativas:

| Restricción | Implicancia |
|---|---|
| **RAM**: 16 GB totales; backend usa ~1 GB en idle y hasta ~10 GB al procesar comprobantes (OCR). | Un modelo STT self-hosted debe caber en el remanente (~4–5 GB en pico). Descarta correr Whisper `large` completo en simultáneo con el OCR. |
| **Cuota gratuita obligatoria**. | Descarta APIs de pago sin free tier. |
| **Procesamiento asíncrono** (no bloquea la respuesta). | La **latencia es deseable pero no un requisito duro**; habilita opciones self-hosted en CPU. |
| **Modelo abierto preferido**. | Favorece la familia Whisper (pesos abiertos, licencia MIT). |
| **No usar `google-genai` (Gemini)** por límites de cuota gratuita. | Excluida. |

## Alternativas evaluadas

Todas las opciones consideradas se basan en **Whisper**, el modelo de STT abierto de
referencia (pesos abiertos, licencia MIT, soporte multilingüe fuerte incluido español).
La diferencia está en **cómo se ejecuta**.

| Opción | Tipo | Cuota gratis | RAM local | Precisión es | Estado |
|---|---|---|---|---|---|
| **faster-whisper `small`** | Self-hosted | Sí (gratis total) | ~1 GB | Alta en montos | **Probado (PoC)** |
| **faster-whisper `large-v3-turbo`** | Self-hosted | Sí (gratis total) | ~2–3 GB | Muy alta | **Probado (PoC)** |
| Whisper (OpenAI, `openai-whisper`) | Self-hosted | Sí (gratis total) | Alta (PyTorch, `large` ~5–10 GB) | Muy alta | Referencia (no probado) |
| Groq (Whisper `large-v3-turbo`) | API gestionada | Sí (free tier) | 0 | Muy alta | No probado |
| Hugging Face Inference Providers | API gestionada | Sí (crédito mensual) | 0 | Muy alta | No probado |
| Cloudflare Workers AI (`whisper-large-v3-turbo`) | API gestionada | Sí (cuota diaria) | 0 | Muy alta | No probado |
| OpenAI Whisper API | API gestionada | **No** (pago por uso) | 0 | Muy alta | Descartada (sin free tier) |
| Gemini (`google-genai`) | API gestionada | Limitada | 0 | Alta | Descartada (decisión de proyecto) |

### faster-whisper (self-hosted) — recomendado

Reimplementación de Whisper sobre **CTranslate2**, hasta ~4x más rápida y con mucho menor
consumo de memoria que el Whisper original, con cuantización (`int8`) que reduce aún más la RAM.

- **Ventajas:** gratis y sin límite de cuota; sin login ni dependencia externa; descarga el
  modelo una vez y corre offline; `int8` en CPU cabe en el presupuesto de RAM; integra como
  microservicio igual que el OCR actual.
- **Desventajas:** consume RAM/CPU local (mitigable eligiendo `small`); los nombres propios
  (bancos) se transcriben con errores y requieren post-proceso.
- **Requisitos:** Python 3.12 (sin wheels de CTranslate2 para 3.14 aún), `faster-whisper`,
  ~0.5 GB (`small`) o ~1.6 GB (`large-v3-turbo`) de descarga de modelo.

### Whisper original (OpenAI, `openai-whisper`)

- **Ventajas:** implementación de referencia, máxima precisión con `large-v3`.
- **Desventajas:** depende de PyTorch (pesado), más lento y mayor RAM que faster-whisper para
  la misma calidad. No aporta sobre faster-whisper en este contexto con RAM acotada.

### Groq / Hugging Face / Cloudflare (APIs gestionadas)

Sirven los **mismos pesos abiertos de Whisper** como API, con free tier, y **0 RAM local**.

- **Ventajas:** elimina el consumo de RAM/CPU local (el motivo original del enfoque API);
  muy rápidas; misma precisión de monto que Whisper.
- **Desventajas:** dependen de conectividad y de la cuota gratuita del proveedor; **Groq**
  presenta fricción de login en Firefox; introducen una dependencia externa para una función
  de captura de datos.
- **Requisitos:** una API key (Groq) o token (HF). HF es la opción más amigable con Firefox;
  Cloudflare Workers AI es una alternativa con cuota diaria gratuita.

## Metodología de la PoC

Ver `stt_poc.ipynb`. Se evaluó faster-whisper (`small` y `large-v3-turbo`, CPU `int8`) sobre:

- **3 audios reales** (`audios/transaccion*.wav`, ~11–16 s) con transcripción de referencia.
- **4 audios sintéticos** (`audios/synthetic/`) generados con `edge-tts` (voces es-AR), con
  *ground truth* exacto y montos variados (incluyendo `15 mil`, `100.200`, `7.899`).

Como **lo crítico es acertar el monto** sin importar cómo se escriba, se reportan tres métricas:

- **Amount OK:** el valor numérico correcto aparece en la transcripción (en letras o dígitos).
  Un número **partido** (`20 238`) cuenta como error; el punto de miles es-AR (`20.238`) es válido.
- **WER normalizado:** WER tras minúsculas, sin acentos, sin muletillas y con números
  canonizados (letras↔dígitos). Mide calidad de transcripción de forma justa.
- **Entities OK:** fracción de bancos esperados detectados.

## Resultados de la PoC

| Modelo | Amount OK | WER norm. | Entities OK | Tiempo/clip (CPU) |
|---|---|---|---|---|
| `small` | **100%** | 0.07 | 75% | **1.8 s** |
| `large-v3-turbo` | **100%** | 0.06 | 58% | 9.8 s |

Transcripciones de los audios reales (modelo `small`):

| Audio | Monto real | Transcripción | Monto OK |
|---|---|---|---|
| transaccion1 | 20238 | "Mandé **20.238** pesos desde Tecabank a mi cuenta de Bluebank." | ✅ |
| transaccion2 | 500 | "Las té **500** pesos en una cosa que compré con mi cuenta de Brubank." | ✅ |
| transaccion3 | 852 | "Desde la cuenta de Lemon, pasé **852** pesos a un amigo." | ✅ |

### Hallazgos

1. **Los montos se transcriben con 100% de acierto** en ambos modelos, sobre audios reales y
   sintéticos. Es la métrica que más importa para la carga por voz.
2. **El formato del número es inconsistente** entre clips: `20.238` (punto de miles), `15 mil`
   (dígito + palabra), `7,899` (coma). Por lo tanto, **es obligatoria una capa de normalización**
   que convierta cualquier representación a un entero, independientemente del modelo elegido.
3. **Los nombres de bancos son poco confiables** (`Brubank→Bluebank/Brewbank`,
   `Lemon→Lehman/Emen`). El STT por sí solo no resuelve nombres propios; deben mapearse por
   **coincidencia difusa contra las cuentas conocidas del usuario** (conjunto finito y pequeño).
4. **`small` vs `large-v3-turbo`:** la precisión de monto es idéntica (100%). `turbo` mejora
   marginalmente el WER (0.06 vs 0.07) pero es ~5x más lento en CPU (9.8 s vs 1.8 s) y usa más
   RAM. Como el procesamiento es asíncrono y el monto ya es 100%, `small` ofrece la mejor
   relación costo/RAM/precisión.

## Requisitos técnicos comparados

| Aspecto | faster-whisper `small` | faster-whisper `large-v3-turbo` | API gestionada (Groq/HF) |
|---|---|---|---|
| RAM local | ~1 GB | ~2–3 GB | 0 |
| Descarga de modelo | ~0.5 GB (una vez) | ~1.6 GB (una vez) | — |
| Latencia (clip corto, CPU) | ~1.8 s | ~9.8 s | < 1 s típico |
| Cuota / costo | Gratis ilimitado | Gratis ilimitado | Free tier (con límites) |
| Login / dependencia externa | No | No | Sí (key/token) |
| Licencia del modelo | MIT (abierto) | MIT (abierto) | MIT (abierto) |
| Encaje con RAM en pico OCR | Sí (10+1 GB) | Ajustado (10+3 GB) | Sí (no usa RAM) |

## Integración con la arquitectura actual

El backend ya aísla la ML pesada en un **microservicio** (`apps/ocr/`) que la API principal
invoca por HTTP, y usa un **flujo de confirmación en dos fases** para los comprobantes. El STT
encaja en ese mismo patrón:

1. El frontend graba el audio (`MediaRecorder`, p. ej. WebM/Opus) y lo envía a la API.
2. Un endpoint encola el trabajo y responde `202 Accepted` (asíncrono, como los comprobantes).
3. Un **servicio STT** (nuevo `apps/stt/` o módulo dentro del microservicio existente) ejecuta
   faster-whisper y devuelve el texto.
4. Una **capa de post-proceso** normaliza el monto (letras/puntos/comas → entero) y resuelve
   banco/cuenta por coincidencia difusa contra las cuentas del usuario.
5. El resultado precarga un **borrador de transacción** que el usuario confirma, reutilizando
   el flujo de confirmación existente.

> Nota: `faster-whisper`/`ctranslate2` aún no tienen wheels para Python 3.14; el servicio STT
> debe correr sobre Python 3.12.

## Recomendación

**Adoptar `faster-whisper` self-hosted (modelo `small`, CPU `int8`) como solución primaria**,
con `large-v3-turbo` como opción de mayor calidad si el presupuesto de RAM lo permite.

Fundamentos:

- **Cumple todas las restricciones:** gratis e ilimitado (cuota), modelo abierto (MIT), sin
  login (evita el bloqueo de Groq en Firefox), y cabe en RAM incluso en el pico de OCR.
- **Precisión suficiente para el caso de uso:** 100% de acierto en montos sobre audios reales
  y sintéticos; el procesamiento asíncrono absorbe la latencia en CPU.
- **Encaje arquitectónico:** replica el patrón de microservicio del OCR ya existente.

**Imprescindible** acompañarlo de dos componentes de post-proceso (independientes del modelo):

1. **Normalización de montos** a entero desde cualquier representación.
2. **Resolución de bancos/cuentas** por coincidencia difusa contra las cuentas del usuario.

**Plan B (si se prioriza 0 RAM local):** una API gestionada que sirva Whisper con free tier —
**Hugging Face Inference Providers** (login amigable con Firefox) o **Cloudflare Workers AI**;
Groq queda como opción a intentar pese a la fricción de login. La precisión de monto sería
equivalente, ya que sirven los mismos pesos de Whisper.

## Próximos pasos

- Validar con audios reales grabados desde el navegador (la PoC usó WAV + TTS).
- Implementar la capa de normalización de montos y la resolución difusa de cuentas.
- Definir si el STT corre en `apps/ocr/` o en un microservicio `apps/stt/` dedicado.
- Medir RAM/latencia concurrente con el OCR en pico antes de fijar `small` vs `large-v3-turbo`.
