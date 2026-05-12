# Bitácora OpenCode — Actividad 3 — Ejercicio 2

**Curso:** Desarrollo Seguro de Software  
**Estudiante:** Viviana Poblete L.  
**Fecha:** 2026-05-11

## Revisión del ADR

Contexto: se le pidió al agente revisar el ADR-005 (Validación de JWT en API Gateway) y preguntar: "¿qué consecuencia negativa estoy ignorando?"

---

### Sugerencia 1 — Token leak por XSS en el cliente

**El agente dice:** "El ADR asume que el token llega seguro al API Gateway, pero no cubre cómo el cliente almacena el token. Si el frontend tiene una vulnerabilidad XSS, un atacante puede robar el token del almacenamiento local (localStorage) antes de que la validación del gateway siquiera lo vea. El ADR mitiga el spoofing en tránsito, pero no el robo en origen."

**Decisión: Acepto**

**Por qué:** Es una consecuencia negativa real que no está documentada. La decisión del ADR es correcta a nivel de gateway, pero el riesgo se desplaza al cliente. La consecuencia debería aparecer en la tabla: el ADR no protege contra token theft vía XSS. Mitigación: cookies HttpOnly + Secure + SameSite en vez de localStorage, o bind del token a la sesión del navegador.

---

### Sugerencia 2 — Dependencia de disponibilidad del IdP

**El agente dice:** "Si el Identity Provider (IdP) cae y la caché JWKS expiró (TTL 300s = 5 min), el gateway no puede validar nuevos tokens. El resultado es denegación de servicio para todos los clientes hasta que el IdP se recupere o se extienda manualmente el TTL de la caché."

**Decisión: Acepto**

**Por qué:** El ADR menciona latencia como consecuencia negativa, pero no la indisponibilidad. Un TTL de 300s significa que 5 minutos después de una caída del IdP, todo el servicio deja de funcionar. Esto debería estar documentado como consecuencia negativa explícita, junto con la estrategia de mitigación (caché extendida en modo degraded, circuit breaker).

---

### Sugerencia 3 — Falta de revocación inmediata de tokens

**El agente dice:** "El ADR rechaza tokens expirados pero no aborda la revocación inmediata. Si un token legítimo se ve comprometido (ej: empleado despedido, dispositivo robado), el token sigue siendo válido hasta su expiración. JWT es stateless por diseño; no hay forma de revocar sin una blacklist, que el ADR no menciona."

**Decisión: Investigo**

**Por qué:** Es una limitación conocida de JWT, pero solucionable. El ADR podría agregar una blacklist distribuida (Redis con TTL) o usar token rotation con refresh tokens de corta duración. La pregunta es si la complejidad vale la pena para el caso de uso actual o si la ventana de expiración (ej: 15 min) es aceptable como riesgo. Fuente a consultar: "JWT Revocation Strategies" en IETF RFC 9068 y OAuth 2.0 Token Revocation (RFC 7009).
