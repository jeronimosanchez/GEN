# GEN — Generación de artefactos conversacionales

**Línea GEN** del sistema de automatización CD — motor adversarial para generar y optimizar artefactos conversacionales (playbooks, examples, intents) antes de desplegarlos.

> Primera instancia: **Petal** (agente de floristería en Dialogflow CX).
> El método es **agnóstico de plataforma**.
> Estado: **en construcción** — arquitectura diseñada, implementación pendiente.

---

## El problema

Optimizar un playbook conversacional a mano es lento y sesgado: el diseñador no ve sus propios puntos ciegos. Iterar directamente en producción es caro e impredecible.

## La idea — Generate → Filter → Validate

```
Fase 1 — GENERACIÓN (LLM1, guiada por el KB)
  gen_context_loader carga el KB (principios + reglas de plataforma + constraints del cliente)
  → LLM1 genera 10-15 variantes YA dentro de bounds
  (cada una explora una elección arquitectónica real: concisión, fallbacks, sintaxis NLU+LLM…)

Fase 2 — CRIBA (barata — reduce antes de gastar en la plataforma)
  · Filtro por razonamiento (LLM2 ≠ generador → sesgo mínimo) → top 3 más diferenciadas
  · Criba behavioral $0 (ADK local) → prueba en local, descarta las débiles (gated por ADK)

Fase 3 — VALIDACIÓN (plataforma real, CX)
  Solo los finalistas corren contra la suite QA en staging
  KPIs por variante vs baseline · la plataforma decide, no el generador
```

**Por qué 10-15 dirigidas > 100 aleatorias:** variantes que exploran dimensiones ortogonales dan señal clara al filtro. 100 variaciones de superficie se agrupan alrededor del original y no descubren nada nuevo.

**Coste estimado por playbook:** ~1,25€ (generación + filtro + validación de 3 finalistas).

---

## Dimensiones de generación

Cada variante explora una **palanca de diseño** distinta — ejemplos: concisión, cobertura de fallbacks, arquitectura NLU+LLM híbrida, una responsabilidad por playbook, few-shot embebidos.

*Lista inicial (del diseño conversacional + plataforma CX), no un set cerrado: **se enriquece a medida que se enriquece el KB** — RES (investigación) y Sistema B (experiencia) alimentan el KB, y el catálogo de palancas crece con él. Se formalizará en `kb_ag_diseño` / `kb_plat_cx`.*

---

## Skills definidas (pendientes de implementar)

| Skill | Rol | Estado |
|---|---|---|
| `gen_context_loader` | Carga y enriquece el contexto antes de generar | 🔴 Definida, no implementada |
| `gen_ag_generator` | Genera 10-15 variantes del artefacto con dimensiones explícitas | 🔴 Definida, no implementada |
| `gen_ag_reviewer` | Filtra y selecciona las 3 mejores variantes | 🔴 Definida, no implementada |
| `gen_ag_adversarial` | Genera conversaciones difíciles para romper el agente | 🔴 Definida, no implementada |

Las definiciones están en `skills/`.

---

## Relación con el sistema

```
CD (kb + método)
    ↓ contexto
   GEN ←→ QAP
    ↓ aprobados
   ACT → Plataforma
```

GEN y QAP se retroalimentan: GEN genera candidatos, QAP valida y devuelve señal de qué mejorar. El ciclo es deliberado — cada iteración produce artefactos más precisos.

**Pre-requisito para arrancar:** staging operativo (las variantes se validan en aislamiento, no en producción).

## El sistema completo

| Repo | Responsabilidad | Estado |
|---|---|---|
| **ACT** ([cx-automation-template](https://github.com/jeronimosanchez/cx-automation-template)) | Despliegue de artefactos a la plataforma | ✅ Operativo |
| **QAP** ([agent-validation-engine](https://github.com/jeronimosanchez/agent-validation-engine)) | Validación estática + dinámica + cribador $0 | ✅ Operativo |
| **GEN** (este) | Generación adversarial de artefactos | 🚧 En construcción |
| **RES** ([RES](https://github.com/jeronimosanchez/RES)) | Investigación continua → knowledge base | 🚧 En construcción |
| **CD** ([CD](https://github.com/jeronimosanchez/CD)) | Hub de conocimiento, método y skills | Existe · en construcción activa |
