# Bitácora OpenCode — Actividad 3 — Ejercicio 1

**Curso:** Desarrollo Seguro de Software  
**Estudiante:** Viviana Poblete L.  
**Fecha:** 2026-05-11

## Revisión del threat model

Contexto: se le pidió al agente (OpenCode) revisar el threat model del agente de reportes construido en sala y señalar amenazas STRIDE/OWASP LLM faltantes.

---

### Sugerencia 1 — LLM03 Supply Chain no está mapeado en la tabla

**El agente dice:** "La tabla de threat model (pág. 42 de la presentación) mapea System Prompt, User Prompt, RAG, MCP, Memoria, Salida del LLM y Costo, pero no incluye una fila para Supply Chain (LLM03). El modelo base, los adaptadores LoRA, las librerías de embedding y el proveedor del LLM son dependencias críticas sin control documentado."

**Decisión: Acepto**

**Por qué:** El threat model cubre 7 componentes de la capa IA pero efectivamente omite Supply Chain. En un sistema que usa un modelo externo (vía API), fine-tunes o adaptadores, un adapter envenenado o un proveedor que cambia su comportamiento son riesgos reales. La defensa (SBOM, hash verification, staging) no está en la tabla. Lo agrego como fila pendiente.

---

### Sugerencia 2 — Privilege escalation por tool chaining

**El agente dice:** "El análisis trata cada tool MCP de forma aislada (tx.list, acct.summary), pero no evalúa combinaciones. Un agente con tools de 'lectura de archivo' + 'envío de email' podría exfiltrar datos sin tener una tool de 'exfiltración' explícita. La composición de permisos no se analiza en el threat model."

**Decisión: Lo investigo más**

**Por qué:** El concepto es válido para sistemas con muchas tools, pero en el agente de reportes actual las tools son solo de lectura (tx.list, acct.summary) y no hay tools de escritura/email en la spec de la clase 2. El riesgo de chaining es bajo en este alcance. Sin embargo, si el sistema evoluciona a más tools (pago de facturas, transferencias), este vector crece. Fuente a consultar: OWASP Agentic AI — Tool Misuse y el principio de "least privilege for tool composition" en el blog de Anthropic sobre agentic security (https://www.anthropic.com/research/agentic-security).

---

### Sugerencia 3 — Spoofing del servidor MCP

**El agente dice:** "No se menciona autenticación mutua (mTLS) ni verificación de identidad del servidor MCP. Un atacante podría interponer un servidor MCP malicioso vía DNS spoofing o compromise del registry de MCP servers, y el agente se conectaría sin validar la identidad del destinatario."

**Decisión: Rechazo**

**Por qué:** El análisis es técnicamente correcto en un contexto general, pero para el alcance de esta actividad (servicio de reportes conversacionales con MCP servers internos en la misma red privada/VPC), el riesgo de spoofing del servidor MCP es bajo. Las defensas propuestas (mTLS) agregan complejidad operativa que no se justifica para el caso actual. Sí corresponde documentarlo como riesgo aceptado o como ADR futuro si los MCP servers se exponen a redes no confiables. Para el threat model de sala, las defensas documentadas (allowlist de servidores, pin de versión, audit log) son suficientes.
