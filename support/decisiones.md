# Decisiones del Proyecto — Pagina Web Sabbia Solutions & Services (Triple S)

**Ultima actualizacion:** 2026-05-22

---

## Como usar este archivo

Cada decision que se toma durante la construccion del alcance y del proyecto queda
registrada aqui con su codigo, contexto, alternativas consideradas y razon.
Las decisiones cerradas no se reabren sin justificacion explicita.

---

## Decisiones cerradas

### D-01 — Identidad visual

**Decision:** Estilo enterprise B2B orientado a datos, inspirado en Palantir / Kinaxis.

| Elemento       | Valor                        |
|----------------|------------------------------|
| Color primario | Azul profundo `#0F1F3D`      |
| Acento         | Cian electrico `#00C2FF`     |
| Neutro claro   | Blanco roto `#F8FAFC`        |
| Neutro oscuro  | Gris `#64748B`               |
| Tipografia     | Inter (sans-serif)           |
| Estilo general | Hero oscuro, metricas visibles, graficas de demanda como elemento visual |

**Alternativas consideradas:** estilo minimalista blanco (descartado — pierde autoridad
tecnica ante directores financieros), estilo colorido startup (descartado — no alinea
con la seriedad B2B).

**Razon:** la audiencia son directores de operaciones, supply chain y financieros.
La paleta oscura con acento cian transmite precision tecnologica y confianza enterprise.

---

### D-02 — Stack tecnologico

**Decision:**

| Capa            | Tecnologia              | Razon                                              |
|-----------------|-------------------------|----------------------------------------------------|
| Framework       | Next.js 14+ (App Router)| SSR/SSG para SEO, API routes para formulario, i18n nativo |
| Lenguaje        | TypeScript              | Tipado en todo el stack, menos errores en runtime  |
| Estilos         | Tailwind CSS v4         | Velocidad de desarrollo, consistencia visual       |
| Animaciones     | Framer Motion           | Scroll animations fluidas, feel enterprise SaaS    |
| ORM             | Prisma                  | Conexion type-safe a PostgreSQL, migraciones simples |
| Base de datos   | PostgreSQL (local)      | Definido por el cliente — almacena leads           |
| Hosting         | Vercel (recomendado)    | Zero-config para Next.js, free tier suficiente     |

**Alternativas consideradas:** Astro (descartado — las API routes para el formulario
son mas complejas), plain HTML (descartado — i18n y formulario requieren mas
infraestructura manual).

**Razon:** Next.js cubre los tres requisitos criticos con una sola tecnologia:
i18n ES/EN, API para guardar leads en PostgreSQL, y SEO con SSR. Es el stack
mas simple que resuelve todos los requisitos reales (PI-2).

---

### D-03 — Idiomas

**Decision:** pagina completamente bilingue Espanol / Ingles usando el sistema de
internacionalizacion nativo de Next.js (App Router + `next-intl` o directorio `/[lang]`).

El usuario selecciona el idioma con un toggle en el header. El idioma por defecto
es Espanol.

**Razon:** la empresa tiene aspiraciones de expansion a mercados de habla inglesa.
Ambos idiomas se gestionan desde el mismo codebase sin duplicar componentes.

---

### D-04 — Almacenamiento de leads

**Decision:** los leads del formulario de contacto se guardan en una tabla `leads`
en PostgreSQL local via Prisma.

Schema de la tabla:

```sql
leads (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name        VARCHAR(255) NOT NULL,
  company     VARCHAR(255) NOT NULL,
  email       VARCHAR(255) NOT NULL,
  phone       VARCHAR(50),
  message     TEXT,
  language    VARCHAR(10) NOT NULL,  -- 'es' o 'en'
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  status      VARCHAR(50) DEFAULT 'new'
)
```

**Alternativas consideradas:** envio por correo (descartado — no persiste),
integracion CRM (diferido — fuera de alcance v1).

**Razon:** el cliente pidio PostgreSQL local para v1. La tabla es suficiente para
gestionar leads manualmente en esta etapa.

---

### D-05 — Estructura de secciones y orden

**Decision:** la pagina tiene las siguientes secciones en este orden:

| # | Seccion              | Proposito                                                    |
|---|----------------------|--------------------------------------------------------------|
| 1 | Hero                 | Tagline fuerte + CTA principal "Solicitar reunion"           |
| 2 | Casos de exito       | Metricas reales de clientes — reduce agotados X%, etc.       |
| 3 | Como funciona        | Proceso en pasos: datos → IA → pronostico → accion           |
| 4 | Que gana la empresa  | Beneficios concretos: inventario, agotados, relacion clientes |
| 5 | Precios              | Planes/tiers de la plataforma                                |
| 6 | Integraciones ERP    | SAP, Oracle, otros ERP del mercado                           |
| 7 | CTA final            | Formulario para solicitar reunion/llamada                    |
| 8 | Footer               | Informacion empresa, links, redes                            |

**Razon:** el orden sigue la logica de persuasion B2B: primero prueba social
(casos de exito), luego comprension (como funciona), luego valor (que gana),
luego friccion de precio, luego confianza tecnica (integraciones), luego accion.

---

### D-06 — CTA principal

**Decision:** la accion principal de la pagina es "Solicitar reunion" o "Agendar llamada".
El formulario captura: nombre, empresa, correo, telefono (opcional), mensaje (opcional).

No hay registro de usuario, no hay trial gratuito, no hay compra en linea.
La conversion es 100% humana — el equipo de Triple S hace el seguimiento.

**Razon:** producto B2B complejo con ciclo de venta consultivo. El cliente pidio
explicitamente que la accion sea solicitar una reunion/llamada.

---

### D-07 — Audiencia objetivo

**Decision:** la pagina habla principalmente a:
- Directores de Operaciones / Supply Chain
- Directores Financieros / CFO
- En menor medida: perfiles tecnicos (CTO, IT Manager)

El tono es: profesional, orientado a resultados de negocio, con respaldo tecnico
sin exceso de jerga. Los casos de exito se expresan en metricas de negocio
(% reduccion de agotados, $ ahorro en inventario), no en metricas tecnicas.

---

### D-08 — Casos de exito

**Decision:** los casos de exito son ficticios pero verosimiles. Se usan empresas
manufactureras colombianas ubicadas en tres ciudades: Medellin, Bucaramanga y
Barranquilla. Cada caso incluye nombre de empresa, ciudad, sector manufacturero,
metrica principal lograda (% reduccion agotados, % mejora precision pronostico, etc.)
y una cita del director responsable.

Los casos de exito se gestionan desde un archivo de contenido, igual que los precios
(ver D-09), para poder actualizarlos sin tocar componentes.

**Razon:** el cliente no tiene casos reales disponibles para v1. Las ciudades colombianas
anclan la credibilidad local ante la audiencia objetivo inicial.

---

### D-09 — Gestion de precios y contenido editable sin codigo

**Decision:** los precios y los casos de exito se gestionan desde archivos JSON en
`content/` en la raiz del proyecto. Los componentes de Next.js leen estos archivos
en tiempo de build — no hay hardcoding de valores en los componentes.

Estructura de archivos de contenido:

```
content/
  pricing.json       ← planes, precios, features por plan
  cases.json         ← casos de exito con empresa, ciudad, metricas, cita
```

Para cambiar un precio: editar `content/pricing.json` y hacer redeploy.
Sin tocar ningun componente React.

Ejemplo de estructura `pricing.json`:
```json
{
  "plans": [
    {
      "id": "starter",
      "name": "Starter",
      "price_usd": 299,
      "billing": "monthly",
      "features": ["Hasta 50 SKUs", "1 empresa cliente", "Soporte email"]
    },
    {
      "id": "gold",
      "name": "Gold",
      "price_usd": 799,
      "billing": "monthly",
      "features": ["Hasta 500 SKUs", "10 empresas cliente", "Soporte prioritario"]
    }
  ]
}
```

**Alternativas consideradas:** CMS headless (descartado — complejidad innecesaria para v1),
valores hardcodeados en componentes (descartado — el cliente pidio explicitamente poder
cambiar precios sin codificacion), variables de entorno (descartado — no es el uso
correcto para contenido de negocio).

**Razon:** cumple el requisito del cliente de cambiar precios sin codigo. Es la
solucion mas simple (PI-2) — un archivo JSON es editable por cualquier persona
sin conocimiento tecnico.

---

### D-15 — Nombre de la empresa

**Decision:** el nombre legal completo es **Sabbia Solutions & Services**.
El nombre corto de uso cotidiano es **Triple S**.

En la pagina web se usa el nombre completo "Sabbia Solutions & Services" como marca
principal. "Triple S" se puede usar como referencia informal pero no como nombre
de marca en titulos, footer, o meta tags.

---

### D-14 — Repositorio GitHub

**Decision:** el repositorio oficial del proyecto es:
`https://github.com/jdrodriguez1000/web_tripleS.git`

El Orchestrator verifica en cada inicio de sesion que el remote esta configurado
correctamente. Sin remote confirmado, el Orchestrator no activa ningun Worker.

Branch strategy (alineada con D-03 de la metodologia):
- `main` — solo codigo con APPROVED
- `sprint/[comp-name]` — trabajo en curso por componente

---

### D-13 — Archivos de persistencia del harness

**Decision:** el harness usa los 5 mecanismos de persistencia de la metodologia.
`support/avance.md` es un archivo separado — es para Claude Code mientras construye
el harness, no para los agentes del harness.

#### Los 5 mecanismos del harness

**1. `claude-progress.txt`** — escribe solo el Orchestrator

```
Responde: que componentes se han construido y cuales faltan?

Formato de texto plano:
COMPLETADOS: hero, cases
EN CURSO: how-it-works (fase: GREEN)
PENDIENTES: value-prop, pricing, integrations, cta, footer
ULTIMO COMMIT: feat(how-it-works): implementacion base responsive
```

**2. `harness-state.json`** — escribe solo el Orchestrator

```json
{
  "project": "triple-s-web",
  "status": "IDLE | IN_PROGRESS | BLOCKED | FAILED",
  "current_component": "hero | cases | how-it-works | value-prop | pricing | integrations | cta | footer",
  "current_phase": "SPEC | RED | GREEN | REFACTOR | EVAL | APPROVED",
  "checkpoints": {
    "hero":          { "status": "APPROVED",     "timestamp": "2026-05-22T10:00:00Z" },
    "cases":         { "status": "IN_PROGRESS",  "timestamp": "2026-05-22T11:00:00Z" },
    "how-it-works":  { "status": "PENDING",      "timestamp": null },
    "value-prop":    { "status": "PENDING",      "timestamp": null },
    "pricing":       { "status": "PENDING",      "timestamp": null },
    "integrations":  { "status": "PENDING",      "timestamp": null },
    "cta":           { "status": "PENDING",      "timestamp": null },
    "footer":        { "status": "PENDING",      "timestamp": null }
  },
  "last_updated": "2026-05-22T11:00:00Z"
}
```

**3. `execution-state.json`** — escribe solo el Worker activo

```json
{
  "component": "cases",
  "agent": "cases_executor",
  "phase": "GREEN",
  "last_checkpoint": "tests-passing",
  "context_usage_pct": 45,
  "last_updated": "2026-05-22T11:30:00Z"
}
```

**4. Git history** — escribe solo el Orchestrator, usando la convencion de D-03
de la metodologia. Prerequisito: repo GitHub configurado antes del primer ciclo.

**5. Handoff artifacts** — en `support/handoffs/[comp]_handoff.md`

```markdown
# Handoff: [comp]

## Paths de artefactos
- Spec: support/specs/[comp]_spec.md
- Tests: src/components/[comp]/[comp].test.tsx
- Implementacion: src/components/[comp]/[comp].tsx
- Evaluacion: support/evals/eval_[comp].json

## Estado
- Fase completada: APPROVED
- Checkpoint: [ultimo checkpoint]
```

**Single Writer Rule:** cada archivo tiene exactamente un dueno.
Elimina condiciones de carrera por diseno.

**Nota sobre avance.md:** `support/avance.md` NO es un archivo del harness.
Es el archivo de orientacion de Claude Code mientras construye el harness.
Los agentes del harness no lo leen ni escriben.

---

### D-12 — Tres roles del harness

**Decision:** cinco agentes con responsabilidad unica e intransferible:

| Agente | Responsabilidad | Prohibido |
|--------|----------------|-----------|
| Orchestrator | Coordinar ciclo, escribir estado, commits, mergear ramas | Escribir codigo de componentes, evaluar outputs |
| `[comp]_specifier` | Producir `spec.md` — comportamiento, casos borde, criterios de aceptacion | Escribir codigo, escribir tests, evaluar su propia spec |
| `[comp]_tester` | Escribir tests en RED contra la spec, sin ver implementacion | Escribir codigo de implementacion, modificar spec, pasar tests a GREEN el mismo |
| `[comp]_executor` | Escribir codigo hasta GREEN, luego REFACTOR | Modificar tests para que pasen, tocar `harness-state.json`, evaluar su propio output |
| Evaluator | Revisar output del Executor contra spec con rubrica fija | Modificar artefactos, ejecutar codigo, proponer soluciones, heredar contexto del Executor |

**Razon:** alineado con D-11 (P1, P3, P4). Tester con contexto limpio produce tests
mas adversariales. Evaluator independiente elimina el sesgo de auto-evaluacion.

---

### D-11 — Principios que gobiernan todos los agentes

**Decision:** los 8 principios aplican sin excepcion a cada agente del harness,
con la siguiente implementacion concreta para este proyecto:

| Principio | Implementacion |
|-----------|----------------|
| P1 — Separacion de roles | Specifier diseña, Tester escribe tests, Executor escribe codigo. Sin cruces. |
| P2 — Artefactos de handoff | Cada agente produce un archivo concreto: `spec.md` → tests → componente implementado |
| P3 — Evaluador externo | Evaluator corre en instancia separada con contexto limpio, nunca hereda historial del Executor |
| P4 — Context resets | Reset proactivo al 70% de uso de tokens. Cada agente arranca con contexto fresco. |
| P5 — Contratos explicitos | Sprint Contract definido antes de activar cualquier agente — "terminado" acordado antes de empezar |
| P6 — Escalamiento proporcional | Hero y CTA usan Opus 4.7 (alta ambiguedad visual/copy); secciones informativas usan Sonnet 4.6 |
| P7 — Herramientas criticas | Evaluator es read-only. Workers no tocan `harness-state.json`. Lista explicita por agente. |
| P8 — Observabilidad | Checkpoints con timestamp en `harness-state.json`. Decisiones del Evaluator en `eval_[comp].json`. |

---

### D-10 — Numero de harnesses

**Decision:** un solo harness — el constructor. No se crea harness operador en v1.

**Razon:** el harness operador no tiene trabajo real que hacer en este proyecto.
Las actualizaciones de contenido (precios, casos) estan resueltas sin agente via
archivos JSON (D-09). La gestion de leads es manual directo a la BD. El deploy
lo maneja Vercel. Agregar un harness operador seria complejidad para casos
hipoteticos — viola E4.

**Condicion para revisar:** si en v2 se agrega blog con CMS, pipeline de actualizacion
automatica de modelos ML, o AB testing de contenido, se evalua separar en dos harnesses.

---

## Log de sesiones

### Sesion 2026-05-22
- Inicio del proyecto
- Creacion de estructura de archivos: `decisiones.md`, `alcance.md`, `avance.md`
- FASE 0 completada — D-01 a D-09 cerradas
- FASE 1.1 completada — D-10 cerrada (un solo harness constructor)
- FASE 1.2 completada — D-11 cerrada (8 principios con implementacion concreta)
- FASE 1.3 completada — D-12 cerrada (5 roles con responsabilidades y prohibiciones)
- FASE 1.4 completada — D-13 cerrada (5 mecanismos de persistencia con schemas exactos)
