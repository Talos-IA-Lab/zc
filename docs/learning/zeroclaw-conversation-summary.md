# Resumen de la conversación: ZeroClaw y plan de aprendizaje

Este documento resume los temas importantes tratados en la conversación sobre el proyecto ZeroClaw y cómo analizarlo como ejercicio de aprendizaje.

---

## 1. Qué es ZeroClaw (para quien no conoce Rust)

- **ZeroClaw** es un **framework de ejecución para agentes de IA**: la infraestructura que permite que un agente (que usa modelos de lenguaje y herramientas) funcione de forma autónoma en distintos entornos.
- Funciona como “motor” que:
  - Recibe mensajes por canales (Telegram, Discord, Slack, etc.).
  - Los envía a un modelo de IA (OpenAI, Anthropic, Ollama, etc.).
  - Ejecuta herramientas (shell, archivos, memoria, navegador).
  - Guarda contexto (memoria).
  - Devuelve respuestas por los mismos canales.
- Todo en **un solo binario** pequeño, rápido y con poco uso de RAM (menos de 5 MB en muchos casos).

### Por qué Rust

- **Rendimiento y tamaño**: un único ejecutable, arranque rápido, poco consumo de memoria; adecuado para Raspberry Pi, VPS baratos o dispositivos con poca RAM.
- **Seguridad**: el lenguaje ayuda a evitar muchos fallos de memoria y condiciones de carrera; importante en un sistema que ejecuta herramientas y se expone a internet.
- **Sin dependencias pesadas**: no requiere intérprete (como Python) ni un runtime enorme; se despliega copiando el binario.

---

## 2. Partes principales del proyecto

| Parte        | Función breve                                                                 |
|-------------|-------------------------------------------------------------------------------|
| **Providers** | Conectores a modelos de IA (OpenAI, Anthropic, Ollama, etc.).               |
| **Channels**  | Por dónde entra y sale la comunicación (Telegram, Discord, Slack, etc.).     |
| **Tools**     | Acciones que el agente puede ejecutar (shell, archivos, memoria, navegador). |
| **Memory**    | Dónde se guarda el contexto (markdown, SQLite, embeddings).                  |
| **Agent**     | Orquesta: recibe mensaje → provider → tools → respuesta por canal.           |
| **Gateway**   | Servidor (p. ej. web) que expone API y dashboard.                             |
| **Security**  | Políticas, allowlists, almacenamiento de secretos.                            |

La arquitectura es **trait-driven**: cada tipo de proveedor, canal, herramienta o memoria implementa un “contrato” (trait); el núcleo depende de esos contratos y se pueden intercambiar implementaciones.

---

## 3. Cómo analizar el código con Cursor

- **Ask / Chat**: hacer preguntas concretas sobre el código; en **modo Ask** el asistente no edita, solo explica.
- **@-mentions**: usar `@Codebase` o `@archivo` para anclar respuestas al código real.
- **Búsqueda**: Cmd/Ctrl+Shift+F para encontrar implementaciones de traits o llamadas a funciones.
- **Go to definition**: F12 o clic derecho sobre tipos/funciones para seguir el flujo.

Orden sugerido de lectura: **entrada (main, lib) → un flujo completo (canal → agente → provider → tools) → traits → un subsistema** (canales, providers, tools, memoria, gateway, seguridad).

---

## 4. Ejercicios propuestos

1. **Seguir un mensaje**: desde la recepción en un canal hasta la llamada al provider.
2. **Añadir un canal**: qué trait implementar, qué archivos tocar, cómo se registra.
3. **Resumir un módulo**: pedir al asistente un resumen de un archivo y sus funciones públicas.
4. **Trazar un comando**: para `zeroclaw channel add telegram ...`, qué módulos intervienen desde `main` hasta persistir la config.

---

## 5. Documentos relacionados

- **Plan de estudio detallado (prompt)**: [zeroclaw-study-plan-prompt.md](zeroclaw-study-plan-prompt.md) — para usarlo como prompt en Cursor y continuar el estudio por fases.
- **Contexto del repo**: `CLAUDE.md` / `AGENTS.md` (protocolo de trabajo y mapa del proyecto).
- **Referencias de runtime**: `docs/commands-reference.md`, `docs/config-reference.md`, etc.
