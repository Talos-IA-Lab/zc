# ZeroClaw — Plan de estudio (prompt para Cursor)

Usa este archivo como **prompt** en Cursor para continuar tu estudio del código. Copia la sección de la fase en la que estés y pégalo en el chat, o di: *"Estoy siguiendo el plan en docs/learning/zeroclaw-study-plan-prompt.md; estoy en Fase X. [Pregunta o petición concreta]."*

---

## Instrucciones para el asistente

Cuando el usuario diga que está siguiendo este plan de estudio de ZeroClaw:

1. Responde en **español**.
2. Prioriza **flujo de datos**, **responsabilidad de cada módulo** y **relación entre módulos**.
3. Cita **archivos y números de línea** cuando sea útil.
4. Si el usuario está en una fase concreta, usa las preguntas y archivos listados en esa fase para guiar la respuesta.
5. No hagas cambios en el código a menos que el usuario pida implementar algo; este plan es para **análisis y aprendizaje**.

---

## Fase 0 — Contexto

**Objetivo:** Tener claro qué hace el proyecto y cómo está organizado.

**Archivos:** `README.md`, `CLAUDE.md` o `AGENTS.md` (secciones 1–4 y 7), `src/lib.rs` (líneas 77–116).

**Preguntas para el asistente:**
- ¿Cuáles son los 5–6 subsistemas principales del proyecto y una frase de qué hace cada uno?
- A partir de `src/lib.rs`, ¿qué módulos son públicos y cuáles están solo para uso interno (`pub(crate)`)?

---

## Fase 1 — Entrada: CLI y arranque

**Objetivo:** Ver cómo arranca el programa y cómo se despachan los comandos.

**Archivos:**
- `src/main.rs`: líneas 219–230 (comando raíz), 227–230 (Commands), 296–302 y 357–375 (Agent y Gateway), y el `match` que despacha comandos (~1090–1165).
- `src/lib.rs`: líneas 119–366 (enums de subcomandos: ServiceCommands, ChannelCommands, MemoryCommands, etc.).

**Preguntas para el asistente:**
- ¿Dónde se parsea la línea de comandos y dónde se despacha cada subcomando (match o if)? Indica números de línea aproximados.
- Para el comando `zeroclaw channel start`, ¿qué función se llama y en qué módulo está?

**Ejercicio:** En `main.rs`, localiza el match que trata `Commands::Channel` y sigue con "Go to definition" la función invocada (ej. `channels::start_channels`).

---

## Fase 2 — Configuración

**Objetivo:** Entender cómo se carga y estructura la configuración que usan el resto de módulos.

**Archivos:**
- `src/config/mod.rs`: primeras ~35 líneas (reexports).
- `src/config/schema.rs`: struct `Config` y secciones (ChannelsConfig, AgentConfig, etc.).

**Preguntas para el asistente:**
- ¿Dónde se define la estructura principal Config y qué secciones principales tiene (nombres de campos)?
- ¿Cómo se carga la config desde disco y qué tipo devuelve?

---

## Fase 3 — Un flujo completo: mensaje → respuesta

**Objetivo:** Seguir un mensaje desde el canal hasta la respuesta (agente, provider, tools).

**Archivos y orden:**
1. `src/channels/traits.rs` — trait `Channel` (send, listen, health_check).
2. `src/channels/mod.rs` — líneas 1–80; lista de canales y exports.
3. Un canal concreto, ej. `src/channels/telegram/mod.rs` — implementación de `Channel`.
4. `src/agent/mod.rs` — exports (`process_message`, `run`, etc.).
5. `src/agent/loop_.rs` — líneas 1–100; función `process_message` o `process_message_with_session`.
6. `src/providers/traits.rs` — trait `Provider` (chat/stream).
7. `src/providers/mod.rs` — líneas 1–45; `create_provider` y lista de providers.
8. `src/tools/traits.rs` — trait `Tool` (name, description, execute).
9. `src/tools/mod.rs` — líneas 1–85; registro de tools (default_tools, all_tools).

**Preguntas para el asistente:**
- ¿Qué métodos debe implementar el trait Channel y qué responsabilidad tiene cada uno?
- ¿Qué hace process_message (o process_message_with_session)? Resumen en 3–5 pasos.
- ¿Qué contrato expone el trait Provider (métodos principales y qué reciben/devuelven)?
- Cuando un mensaje llega por un canal, ¿en qué orden se usan Channel → Agent → Provider → Tools? Indica archivos y funciones clave.

**Ejercicio:** Elegir un canal (ej. Telegram). Desde `start_channels` en `channels/mod.rs`, seguir con "Go to definition" hasta ver cómo se registra un listener que llama a process_message; anotar las funciones del camino.

---

## Fase 4 — Contratos (traits) con detalle

**Objetivo:** Entender las interfaces del sistema sin entrar en todas las implementaciones.

**Archivos (una sesión por archivo si se desea):**

| # | Archivo | Enfocarse en |
|---|---------|--------------|
| 1 | `src/channels/traits.rs` | Trait `Channel` y tipos de mensajes/opciones. |
| 2 | `src/providers/traits.rs` | Trait `Provider`; `ChatMessage`, `ChatRequest`, `ToolCall`, respuestas. |
| 3 | `src/tools/traits.rs` | Trait `Tool`; `ToolResult`; esquema de parámetros. |
| 4 | `src/memory/traits.rs` | Trait `Memory`; categorías; almacenar/recuperar. |

**Preguntas para el asistente:**
- Explica los tipos ChatMessage, ChatRequest, ToolCall y cómo se usan en el flujo del agente.
- ¿Qué debe devolver execute() de un Tool y cómo se pasa ese resultado al modelo?
- ¿Qué operaciones expone Memory y qué es MemoryCategory?

---

## Fase 5 — Subsistemas (elegir uno o varios)

Profundizar por tema: abrir el `mod.rs` del subsistema y un archivo concreto de implementación.

### 5.1 Canales

**Archivos:** `src/channels/mod.rs`, `src/channels/telegram/mod.rs` (o discord, slack).

**Pregunta:** ¿Cómo se construye el TelegramChannel y cómo se conecta listen() con el agente (qué función del agent se llama)?

### 5.2 Providers

**Archivos:** `src/providers/mod.rs`, `src/providers/openai.rs` o `anthropic.rs`.

**Pregunta:** ¿Cómo se traduce una ChatRequest a la API de OpenAI y cómo se mapea la respuesta a ChatResponse / ToolCall?

### 5.3 Tools

**Archivos:** `src/tools/mod.rs`, `src/tools/shell.rs` o `src/tools/file_read.rs`.

**Pregunta:** ¿Cómo se validan los argumentos y qué hace execute() antes de ejecutar el comando?

### 5.4 Memoria

**Archivos:** `src/memory/mod.rs`, `src/memory/traits.rs`, `src/memory/sqlite.rs` o `src/memory/markdown.rs`.

**Pregunta:** ¿Dónde se persisten las entradas y cómo se integra con el agente (quién llama a Memory)?

### 5.5 Gateway

**Archivos:** `src/gateway/mod.rs` (líneas 1–60), `src/gateway/api.rs` si existe.

**Pregunta:** ¿Qué rutas HTTP expone el gateway y cuál recibe mensajes para el agente?

### 5.6 Seguridad

**Archivos:** `src/security/mod.rs`; buscar `SecurityPolicy` y pairing/allowlist.

**Pregunta:** ¿Dónde se aplica SecurityPolicy al ejecutar herramientas y qué comprueba antes de permitir una acción?

---

## Cómo usar este plan

- **Por fases:** Completa Fase 0, luego 1, 2, 3, 4 y 5 (o las que quieras).
- **En Cursor:** Pega el bloque de "Preguntas" de la fase actual o escribe: *"Estoy en Fase X del plan docs/learning/zeroclaw-study-plan-prompt.md; [pregunta concreta]."*
- **Navegación:** Usa "Go to definition" (F12) en tipos y funciones que cite el asistente.
- **Búsqueda:** Cmd/Ctrl+Shift+F para buscar nombres de funciones o `impl ... for ...`.

Cuando termines una fase, repasa mentalmente el flujo antes de pasar a la siguiente.
