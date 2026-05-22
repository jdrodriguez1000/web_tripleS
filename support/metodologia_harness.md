# Metodologia Harness — Construccion Agentica con SDD+TDD

**Ultima actualizacion:** 2026-05-22

---

## Por que existe el harness

Los modelos de lenguaje son irreduciblemente probabilisticos. No puedes eliminar esa
naturaleza — es como funciona el modelo a nivel matematico.

Lo que si puedes hacer es **reducir el espacio de decisiones** donde la probabilidad opera.

```
Sin harness:    El agente decide TODO — que hacer, como hacerlo,
                cuando terminar, si el output esta bien hecho.
                Superficie probabilistica: maxima.

Con harness:    El agente decide SOLO como ejecutar una tarea acotada.
                Que hacer, cuando terminar y si el output esta bien hecho
                ya estan definidos por el contrato y el evaluador.
                Superficie probabilistica: minima.
```

Cada componente del harness elimina una fuente especifica de incertidumbre:

| Componente | Que incertidumbre elimina |
|---|---|
| Sprint Contract (P5) | El agente no decide que es "terminado" — ya esta definido |
| Checkpoints canonicos (E5) | El agente no decide cuando avanzar — el Orchestrator lo verifica |
| Evaluador externo (P3) | El agente no evalua su propio output — un juez independiente con rubrica fija lo hace |
| Herramientas explicitas (P7) | El agente no improvisa como acceder a datos — tiene exactamente las herramientas que necesita |
| Artefactos de handoff (P2) | El agente no interpreta que necesita la siguiente etapa — hay un schema definido |
| Context resets (P4) | El agente no acumula sesgo de conversacion larga — cada etapa empieza fresco |

La formulacion correcta no es **deterministico vs. probabilistico**.
Es **varianza controlada**: el harness reduce la varianza del output al minimo posible
dado un modelo probabilistico. Un agente bien encuadrado con contrato claro, herramientas
precisas y evaluador calibrado produce outputs consistentes y predecibles — no porque el
modelo sea deterministico, sino porque el espacio donde puede fallar se ha reducido
sistematicamente.

---

## Proposito de este documento

Paso a paso para construir un harness agentico desde cero, alineado con los
8 principios y 12 estandares de `support/principios.md`.
Aplica a cualquier proyecto de software — desde una landing page hasta una plataforma ML.
El ejemplo concreto de referencia es la plataforma de pronostico de demanda B2B.

---

## Principios y estandares de referencia

| Codigo | Nombre corto |
|---|---|
| P1 | Separacion de roles |
| P2 | Artefactos de handoff |
| P3 | Evaluador externo independiente |
| P4 | Context resets sobre compactacion |
| P5 | Contratos explicitos antes de ejecutar |
| P6 | Escalamiento proporcional |
| P7 | Herramientas como extensiones criticas |
| P8 | Observabilidad y trazabilidad |
| E1 | Persistencia de estado entre sesiones |
| E2 | Context Anxiety — cuando hacer reset |
| E3 | Calibracion del evaluador |
| E4 | Minima complejidad + evolucion continua |
| E5 | Ejecucion durable — reanudar desde checkpoint |
| E6 | Outputs al filesystem, no al orquestador |
| E7 | Paralelizacion explicita |
| E8 | Extended Thinking para reasoning complejo |
| E9 | Evaluacion temprana con muestras pequenas |
| E10 | Secuencia de inicio de sesion estructurada |
| E11 | Busqueda de amplio a estrecho |
| E12 | Arquitectura Orquestador-Trabajador |

---

## FASE 0 — Entender el dominio (prerequisito del harness)

El harness no se diseña en paralelo con el dominio. Estas decisiones se toman y se
cierran antes de diseñar un solo agente. Si cambias el dominio despues, reescribes
los contratos.

### 0.1 Entender el problema de negocio

**Que defines:** que produce la plataforma, para quien, que decisiones habilita, que
datos entran y que outputs se producen.

**Por que es prerequisito:** sin esto no sabes cuantas etapas tiene el pipeline
ni que responsabilidad tiene cada agente.

**Principio aplicado — E11 (amplio a estrecho):** empieza con preguntas amplias
(que problema resuelve el negocio, quien lo usa) antes de profundizar en detalles
tecnicos. No comprometas el diseño a una solucion tecnica antes de entender el
espacio del problema.

### 0.2 Definir el pipeline

**Que defines:** cuantas etapas tiene el proceso, que produce cada etapa, que necesita
cada etapa para funcionar, dependencias entre etapas.

**Por que es prerequisito:** cada etapa mapea a un agente. Sin el pipeline no sabes
cuantos agentes necesitas ni en que orden van.

**Principio aplicado — P1 (separacion de roles):** cada etapa debe tener una
responsabilidad unica y acotada. Si una etapa hace dos cosas distintas, son dos
etapas. No diseñes etapas todopoderosas — violan P1 desde el inicio.

**Principio aplicado — E4 (minima complejidad):** diseña el pipeline mas simple que
resuelva el problema real. No agregues etapas para casos hipoteticos futuros. Tres
etapas que funcionan son mejor que nueve a medio diseñar.

### 0.3 Definir el modelo de datos

**Que defines:** tablas, campos, relaciones, multi-tenant, PKs, estrategia de
artefactos (BD vs. storage).

**Por que es prerequisito:** los agentes necesitan saber que leer y donde escribir
antes de poder definir sus herramientas (P7). Sin modelo de datos los contratos
quedan incompletos.

### 0.4 Definir el stack tecnologico

**Que defines:** backend, base de datos, librerias ML, frontend, infraestructura.

**Por que es prerequisito:** determina que herramientas son disponibles para cada
agente. No puedes definir herramientas (P7) sin saber el stack.

**Principio aplicado — E4 (minima complejidad):** elige el stack mas simple que
cubra los requisitos reales. Cada tecnologia adicional es un agente que necesita
conocerla o una herramienta mas que mantener.

### 0.5 Definir criterios de evaluacion temprana

**Que defines:** ~20 casos representativos del pipeline con input conocido y output
esperado. Estos casos se usan para calibrar el evaluador y para validar el tracer
bullet antes de construir todos los agentes.

**Por que es prerequisito:** sin criterios de evaluacion definidos antes de construir,
no tienes forma de saber si lo que construiste funciona correctamente.

**Principio aplicado — E9 (evaluacion temprana):** los cambios tempranos tienen
efectos dramaticos en calidad (diferencias de 30% a 80%). Definir los casos de
prueba antes de escribir codigo garantiza que la construccion apunta al target correcto.
No esperes a tener el harness completo para evaluar.

---

## FASE 1 — Diseñar el harness

Con el dominio cerrado, diseñas la arquitectura agentica completa antes de escribir
una linea de codigo.

### 1.1 Decidir cuantos harnesses y con que proposito

**Que defines:** si el proyecto necesita un harness constructor (para construir el
software), un harness operador (para operar el sistema en produccion), o ambos.
Las diferencias en contexto de trabajo, herramientas, trigger y evaluador determinan
si deben ser sistemas separados.

**Principio aplicado — P1 (separacion de roles):** un harness que construye y opera
al mismo tiempo es un agente todopoderoso. Si los propositos son distintos, los
harnesses deben ser distintos.

### 1.2 Definir los principios que gobiernan todos los agentes

**Que defines:** los principios que aplican a cada agente del harness sin excepcion.
Deben quedar documentados como contrato de diseño, no como sugerencias.

Los principios minimos que todo harness debe implementar:

| Principio | Como se implementa en diseño |
|---|---|
| P1 — Separacion de roles | Un rol por agente. Nadie hace de todo. Definir que esta prohibido por rol. |
| P2 — Artefactos de handoff | Cada agente recibe un artefacto de entrada y produce uno de salida. |
| P3 — Evaluador externo | Quien genera no evalua. El evaluador es un agente separado e independiente. |
| P4 — Context resets | Contexto limpio por agente. No acumular historial entre etapas. |
| P5 — Contratos explicitos | Cada agente tiene su Sprint Contract definido antes de ejecutarse. |
| P6 — Escalamiento proporcional | Definir nivel de esfuerzo por agente: 1 worker, N workers, o extended thinking. |
| P7 — Herramientas criticas | Herramientas permitidas y prohibidas por agente, igual de importantes que el prompt. |
| P8 — Observabilidad | Cada decision de cada agente es auditable. Estructura del log definida desde el diseño. |

### 1.3 Definir los tres roles del harness

**Que defines:** Orchestrator, Worker y Evaluator con sus responsabilidades, lo que
pueden hacer y lo que tienen prohibido.

**Principio aplicado — P1 (separacion de roles):**

| Rol | Responsabilidad unica | Prohibido |
|---|---|---|
| Orchestrator | Coordinar, mediar transiciones, escribir estado | Ejecutar trabajo del dominio, evaluar outputs |
| Worker | Ejecutar la tarea del dominio (spec, test, dev, inferencia) | Modificar harness-state.json, evaluar su propio output |
| Evaluator | Revisar output del Worker contra la spec con rubrica | Modificar artefactos, ejecutar codigo, proponer soluciones |

**Principio aplicado — E12 (Orquestador-Trabajador):** el Orchestrator analiza el
objetivo y guarda su plan en memoria ANTES de crear subagentes. Si el contexto crece
y el Orchestrator pierde el plan, no puede coordinar. Los Workers operan con ventanas
de contexto propias y frescas — nunca heredan el historial del Orchestrator.

### 1.4 Definir los archivos de persistencia con sus schemas exactos

**Que defines:** los 5 archivos de persistencia, quien escribe cada uno, quien lee
cada uno y el schema exacto de cada campo.

**Principio aplicado — E1 (persistencia de estado):** sin archivos de persistencia
cada sesion comienza ciega. El harness desperdicia contexto reorientandose en lugar
de ejecutar trabajo real.

**Principio aplicado — E6 (outputs al filesystem):** los agentes escriben sus outputs
directamente al filesystem. El Orchestrator recibe solo paths, nunca el contenido
completo. Esto evita la degradacion de informacion al pasar por multiples agentes
("telefono descompuesto") y permite auditoria directa.

| Archivo | Escritor (unico) | Pregunta que responde |
|---|---|---|
| `claude-progress.txt` | Orchestrator | Que se ha hecho y que falta? |
| `harness-state.json` | Orchestrator | En que estado esta el sistema ahora mismo? |
| `execution-state.json` | Workers | Que estan haciendo los workers ahora mismo? |
| Git history | Orchestrator | Que cambio, cuando y por que? |
| Handoff artifact | Orchestrator | Que necesita saber la fase siguiente? |

**Single Writer Rule:** cada archivo tiene exactamente un dueno. Elimina condiciones
de carrera por diseño, no por codigo.

**Principio aplicado — E5 (ejecucion durable):** el schema de `harness-state.json`
debe incluir los checkpoints canonicos con nombres exactos. Esos nombres son la fuente
de verdad para reanudar desde el punto de fallo — no desde el inicio.

### 1.5 Definir el Sprint Contract de cada agente

**Que defines:** para cada agente del pipeline, su contrato completo antes de escribir
una linea de codigo.

**Principio aplicado — P5 (contratos explicitos):** acordar "terminado" antes de
empezar. Sin contrato, el Worker decide que es suficiente. Con contrato, el Evaluator
puede verificar objetivamente.

Cada Sprint Contract debe incluir:

| Campo | Descripcion |
|---|---|
| Rol y proposito | Una linea — que hace y que NO hace este agente |
| Input | Artefactos de entrada con paths y schemas |
| Output | Artefactos de salida con paths y schemas |
| Herramientas permitidas | Lista explicita — P7 |
| Herramientas prohibidas | Lista explicita — P7 |
| Checkpoints canonicos | Nombres exactos de cada checkpoint — E5 |
| Criterios de aceptacion | Uno por criterio, cada uno mapea 1:1 a un test |
| Reglas de comportamiento | Lo que el agente debe y no debe hacer |
| Trigger de context reset | En que porcentaje de uso de tokens hace reset — E2 |
| Politica de escalamiento | Cuantos workers, si usa extended thinking — P6 |

**Principio aplicado — E5 (ejecucion durable):** los checkpoints se definen en el
Sprint Contract, no los decide el Worker durante la ejecucion. El Worker no puede
mover el arco de meta.

**Principio aplicado — E2 (context anxiety):** definir en el contrato el trigger
proactivo de reset (70% de uso de tokens recomendado) y el trigger reactivo (el
agente empieza a cerrar tareas sin completarlas o a saltarse pasos). El reset agrega
complejidad orquestal pero preserva la calidad del output.

### 1.6 Definir la politica de escalamiento por agente

**Que defines:** para cada agente, el nivel de esfuerzo proporcional a su complejidad.
Dos ejes independientes: horizontal (cuantos workers en paralelo) y vertical (si usa
extended thinking y con que budget).

**Principio aplicado — P6 (escalamiento proporcional):**

| Nivel | Cuando aplica | Como se implementa |
|---|---|---|
| 1 Worker, sin thinking | Tareas mecanicas y deterministicas | Un solo Worker, sin budget de thinking |
| 1 Worker, thinking moderado | Razonamiento complejo pero acotado | Budget 6,000 tokens |
| 1 Worker, thinking alto | Decisiones criticas con alta ambiguedad | Budget 12,000 tokens |
| N Workers en paralelo | Tareas independientes entre si | Max 5 simultaneos — E7 |
| N Workers + thinking | Tareas independientes y complejas | Combinar ambos ejes |

**Principio aplicado — E7 (paralelizacion):** identificar en diseño cuales fases
del pipeline son independientes entre si y pueden correr en paralelo. La paralelizacion
no es una optimizacion — es una decision de arquitectura que debe tomarse antes de
construir.

**Principio aplicado — E8 (extended thinking):** reservar para los pasos donde la
calidad del razonamiento es critica. No aplicar de forma indiscriminada. El budget
de thinking loggeado pero no propagado al siguiente agente.

### 1.7 Definir la politica de herramientas por agente

**Que defines:** para cada agente, la lista explicita de herramientas permitidas y
prohibidas. Las herramientas son tan criticas como el prompt.

**Principio aplicado — P7 (herramientas criticas):** un agente con herramientas
incorrectas produce outputs incorrectos aunque el prompt sea perfecto. Definir las
herramientas en diseño, no despues de que el agente falle por no tenerlas.

Reglas minimas:
- El Evaluator es read-only — nunca tiene herramientas de escritura
- Los Workers no tienen acceso a `harness-state.json` — ese archivo es del Orchestrator
- Las herramientas prohibidas se documentan explicitamente, no se omiten en silencio

### 1.8 Definir la estrategia de observabilidad

**Que defines:** que se loggea, en que formato, donde se almacena y como se audita
cada decision de cada agente.

**Principio aplicado — P8 (observabilidad):** la trazabilidad no es una feature
adicional — es un requisito del harness desde el primer dia. Sin traces completos
no puedes depurar por que un agente tomo una decision incorrecta.

Minimo necesario:
- Cada checkpoint alcanzado queda registrado con timestamp en `harness-state.json`
- Cada decision del Evaluator queda en `eval_[comp].json` con razon explicita
- El git history es el log de auditoria de cambios — commits descriptivos al finalizar
  cada sesion (E1)
- El thinking de extended thinking se loggea pero no se propaga (E8)

### 1.9 Calibrar el evaluador antes de construir

**Que defines:** la rubrica de evaluacion con scores 0.0-1.0 por dimension, los
pesos de cada dimension, los umbrales de aprobacion y los ejemplos few-shot que
calibran al evaluador antes de su primer uso real.

**Principio aplicado — E3 (calibracion del evaluador):** sin calibracion explicita,
los evaluadores son sistematicamente lenientes incluso con outputs de baja calidad.
La calibracion se hace en diseño, no cuando el evaluador ya rechaza (o no rechaza)
outputs en produccion.

Rubrica minima:

| Dimension | Peso | Descripcion |
|---|---|---|
| Cobertura de spec | 0.35 | Cada criterio de aceptacion tiene un test que lo verifica |
| Tests en verde | 0.35 | Todos los tests pasan — binario: un fallo es rechazo automatico |
| Restricciones | 0.20 | El agente respeta las reglas de comportamiento del contrato |
| Scope | 0.10 | La implementacion no agrega ni omite comportamiento fuera del contrato |

Umbrales: score global >= 0.80 + piso por dimension >= 0.70.
"Tests en verde" es binario — si falla uno, RECHAZA sin importar el score global.

Few-shot: proveer al evaluador al menos 2 ejemplos con desglose de puntajes detallados
(uno que aprueba, uno que rechaza) antes de activarlo en el primer componente real.

### 1.10 Definir los triggers y el scheduling

**Que defines:** como arranca el pipeline (manual, cron, evento), como se reanuda
ante un fallo, como se previenen runs concurrentes, como se gestiona la aprobacion
humana cuando es necesaria.

**Principio aplicado — E5 (ejecucion durable):** cada trigger debe tener definido
su punto de entrada exacto y su comportamiento ante estado BLOCKED o FAILED. Un
trigger que ignora el estado del sistema puede causar runs concurrentes o sobrescribir
trabajo en curso.

### 1.11 Definir la secuencia de inicio de sesion

**Que defines:** el ritual de arranque exacto para cada tipo de sesion: inicio de
proyecto, continuacion de sesion interrumpida y inicio de nuevo ciclo.

**Principio aplicado — E10 (secuencia de inicio):** cada harness debe definir una
secuencia de arranque explicita. Sin ella, cada sesion comienza con el agente
reorientandose en lugar de ejecutando trabajo real.

Secuencia minima para cualquier harness:

```
Inicio (primera sesion):
1. Verificar directorio y ambiente
2. git log — confirmar repo limpio
3. Leer claude-progress.txt — confirmar que el componente no existe ya
4. Leer harness-state.json — confirmar status IDLE
5. Activar el primer agente del ciclo

Continuacion (sesion interrumpida):
1. Leer harness-state.json — identificar checkpoint donde quedo
2. Leer execution-state.json — identificar que estaba haciendo el worker
3. Leer handoff del checkpoint actual — reconstruir contexto minimo (solo paths)
4. Activar el agente correcto segun el checkpoint
```

**Principio aplicado — E6 (outputs al filesystem):** el briefing de resumption
incluye solo paths — nunca el contenido de los artefactos. El Worker lee los archivos
directamente del filesystem.

### 1.12 Definir el sistema de metricas

**Que defines:** como sabes que el harness esta funcionando bien. Dos tipos: metricas
por agente (Tipo 1) y metricas por harness (Tipo 2). Tres capas de madurez con
condicion de activacion.

**Principio aplicado — E4 (evolucion continua):** el harness no es estatico. Las
metricas son el mecanismo que activa la evolucion: si un componente sistematicamente
produce rechazos, el contrato o la rubrica necesitan ajuste. Si un componente nunca
falla, puede que el evaluador este siendo demasiado leniente.

| Capa | Cuando se activa | Que mide |
|---|---|---|
| Capa 0 | Desde el primer Sprint Contract | Completitud, calidad de output, uso de herramientas, eficiencia |
| Capa 1 | Primer ciclo completo real | Cobertura de datos, calidad de features, MAPE/RMSE, calidad de handoffs |
| Capa 2 | Despues de 2 ciclos completos | Degradacion, reentrenamiento, confiabilidad, velocity |

---

## FASE 2 — Construir el harness

Con todo diseñado y documentado, construyes en orden de dependencia.

### 2.1 Persistencia

**Que construyes:** los schemas de `harness-state.json`, `execution-state.json` y
los handoff artifacts. Los archivos vacios con estructura valida listos para ser
escritos por los agentes.

**Por que va primero:** sin persistencia ningun agente puede guardar ni leer estado.
Es la infraestructura que habilita E1, E5 y E6.

**Principio aplicado — E4 (minima complejidad):** implementar el schema minimo que
cubra los checkpoints definidos en diseño. No agregar campos para casos hipoteticos.
Un campo extra no usado es deuda de mantenimiento.

### 2.2 Orchestrator minimal

**Que construyes:** el Orchestrator con exactamente las capacidades necesarias para
gestionar un ciclo completo del primer componente. Sin features adicionales.

**Principio aplicado — E12 (Orquestador-Trabajador):** el Orchestrator guarda su
plan en `harness-state.json` ANTES de activar cualquier Worker. Si el contexto crece
y el Orchestrator pierde el plan, lo recupera leyendo el archivo — no desde memoria.

**Principio aplicado — E4 (minima complejidad):** construir el Orchestrator minimal
primero. Agregar capacidades solo cuando un ciclo real demuestre que son necesarias.
Un Orchestrator sobre-diseñado antes de tener Workers funcionando es deuda prematura.

### 2.3 Tracer bullet — un ciclo completo end-to-end

**Que construyes:** el componente mas simple del pipeline, completo desde spec hasta
APPROVED, usando el ciclo SDD+TDD con evaluador independiente.

**Por que va aqui:** el tracer bullet prueba que la arquitectura del harness es correcta
antes de invertir en construir todos los agentes. Un error de diseño descubierto en el
tracer bullet se corrige en horas — el mismo error descubierto en el agente 7 se
corrige en dias.

**Principio aplicado — E9 (evaluacion temprana):** evaluar el tracer bullet con los
~20 casos representativos definidos en 0.5 ANTES de construir los agentes restantes.
Los cambios tempranos tienen efectos dramaticos en calidad. Si el tracer bullet falla
sistematicamente, hay un problema de diseño que debes resolver antes de continuar.

**Principio aplicado — E4 (minima complejidad):** el tracer bullet es deliberadamente
simple. No implementes el caso mas complejo del componente — implementa el caso base
que prueba el ciclo completo. La complejidad se agrega despues de que el ciclo funciona.

### 2.4 Evaluador calibrado

**Que construyes:** el Evaluator como agente independiente con la rubrica y los
ejemplos few-shot definidos en 1.9.

**Por que va aqui y no antes:** el Evaluator necesita artefactos reales para calibrarse.
Los outputs del tracer bullet son los primeros artefactos reales disponibles.

**Principio aplicado — E3 (calibracion):** activar el Evaluator con los few-shot
ejemplos sobre los outputs del tracer bullet como primer caso de calibracion. Verificar
que el score es consistente con la expectativa humana antes de usarlo en los agentes
siguientes.

**Principio aplicado — P3 (evaluador externo):** el Evaluator corre en una instancia
separada con contexto limpio. Nunca hereda el historial del Worker que evalua.

### 2.5 Agentes restantes en orden de dependencia

**Que construyes:** cada agente del pipeline en el orden que dictan sus dependencias
de datos. Cada agente sigue el mismo ciclo: spec → RED → GREEN → REFACTOR → APPROVED.

**Principio aplicado — P2 (artefactos de handoff):** cada agente produce exactamente
un handoff que el siguiente agente consume. El handoff define el contrato entre etapas.
Si el handoff cambia, el contrato entre agentes debe renegociarse.

**Principio aplicado — E5 (ejecucion durable):** cada agente implementa resumption
desde su ultimo checkpoint. Un fallo a mitad de un agente no reinicia el ciclo completo
— reinicia desde el ultimo checkpoint guardado en `execution-state.json`.

**Principio aplicado — E7 (paralelizacion):** los agentes cuyas fases internas son
independientes pueden paralelizar Workers (max 5 simultaneos). Esta decision ya fue
tomada en diseño (1.6) — aqui solo se implementa.

**Principio aplicado — E8 (extended thinking):** los agentes con budget de thinking
definido en 1.6 lo implementan aqui. El thinking se loggea en el artefacto de output
pero no se pasa al siguiente agente.

### 2.6 Triggers y scheduling

**Que construyes:** los endpoints y jobs que activan el pipeline segun los tipos de
trigger definidos en 1.10: manual, cron, evento de degradacion, aprobacion humana.

**Por que va aqui:** los triggers solo tienen sentido cuando los agentes ya funcionan.
Un trigger que llama a un agente roto no aporta nada durante la construccion.

**Principio aplicado — E5 (ejecucion durable):** cada trigger verifica el estado del
sistema antes de activar. Si el status es IN_PROGRESS, BLOCKED o FAILED, el trigger
rechaza con codigo de error explicito — no sobreescribe trabajo en curso.

### 2.7 Metricas

**Que construyes:** los colectores de metricas Tipo 1 (por agente) y Tipo 2 (por
harness) segun las capas de madurez definidas en 1.12.

**Por que va al final:** antes de tener datos reales de ciclos completos no hay
nada util que medir. Instrumentar antes de tener datos es infraestructura sin retorno.

**Principio aplicado — E4 (evolucion continua):** las metricas son el insumo para
evolucionar el harness. Despues de cada ciclo real, revisar: hay componentes que
sistematicamente producen rechazos? Hay componentes que nunca fallan y cuyo evaluador
podria estar siendo leniente? Remover un componente a la vez y medir el impacto antes
de decidir si se mantiene.

---

## Resumen del orden completo

```
FASE 0 — PREREQUISITOS (en orden, cada uno depende del anterior)
    0.1  Problema de negocio          → E11
    0.2  Pipeline ML                  → P1, E4
    0.3  Modelo de datos              → P7
    0.4  Stack tecnologico            → E4
    0.5  Criterios de evaluacion      → E9

FASE 1 — DISEÑO (en orden, cada uno depende del anterior)
    1.1  Cuantos harnesses            → P1
    1.2  Principios que gobiernan     → P1-P8
    1.3  Tres roles del harness       → P1, E12
    1.4  Archivos de persistencia     → E1, E6
    1.5  Sprint Contracts             → P5, E2, E5
    1.6  Escalamiento por agente      → P6, E7, E8
    1.7  Herramientas por agente      → P7
    1.8  Observabilidad               → P8, E1
    1.9  Calibracion del evaluador    → E3
    1.10 Triggers y scheduling        → E5
    1.11 Secuencia de inicio          → E10, E6
    1.12 Sistema de metricas          → E4

FASE 2 — CONSTRUCCION (en orden de dependencia)
    2.1  Persistencia                 → E1, E5, E6, E4
    2.2  Orchestrator minimal         → E12, E4
    2.3  Tracer bullet                → E9, E4
    2.4  Evaluador calibrado          → E3, P3
    2.5  Agentes restantes            → P2, E5, E7, E8
    2.6  Triggers y scheduling        → E5
    2.7  Metricas                     → E4
```

---

## Aplicacion agil — que debe estar completo antes de construir

La Fase 1 no es monolitica. Hay pasos que son prerequisitos duros antes de escribir
una linea de codigo, y pasos que se completan just-in-time por agente a medida que
avanza la construccion.

### Prerequisitos duros — cerrar antes de construir nada

| Pasos | Por que no se pueden diferir |
|---|---|
| Fase 0 completa | El dominio no puede cambiar sin reescribir los contratos |
| 1.1 Cuantos harnesses |決ision estructural — cambiarlo despues rompe todo |
| 1.2 Principios que gobiernan | Contrato de diseño que aplica a cada agente sin excepcion |
| 1.3 Tres roles del harness | Sin roles definidos no se pueden diseñar contratos ni herramientas |
| 1.4 Archivos de persistencia | Sin schemas de persistencia ningun agente puede guardar ni leer estado |

### Just-in-time — se completan por agente, al momento de construirlo

| Pasos | Cuando se hacen |
|---|---|
| 1.5 Sprint Contract | Solo para el agente que se va a construir a continuacion |
| 1.6 Escalamiento | Solo para el agente que se va a construir a continuacion |
| 1.7 Herramientas | Solo para el agente que se va a construir a continuacion |
| 1.8 Observabilidad | La estructura general se define al inicio; los detalles por agente se completan just-in-time |
| 1.9 Calibracion del evaluador | Requiere artefactos reales del tracer bullet — imposible antes de 2.3 |
| 1.10 Triggers y scheduling | Irrelevante hasta tener agentes funcionando |
| 1.11 Secuencia de inicio | Se puede refinar despues del primer ciclo real |
| 1.12 Sistema de metricas | Capa 0 al inicio; Capa 1 despues del primer ciclo; Capa 2 despues de dos ciclos |

### Ciclo de trabajo resultante

```
Fase 0 completa
Fase 1.1–1.4 completa
    ↓
Diseñar agente 1 (1.5–1.7)
Construir tracer bullet (2.1–2.3)
Calibrar evaluador (1.9 + 2.4)
    ↓
Diseñar agente 2 (1.5–1.7)
Construir agente 2 (2.5)
    ↓
... iterar por agente
    ↓
Triggers (1.10 + 2.6)
Metricas (1.12 + 2.7)
```

Este ciclo garantiza que las decisiones fundacionales esten cerradas antes de construir,
mientras evita el costo de diseñar contratos detallados para agentes que aun no se van
a construir — contratos que podrian necesitar revision al llegar al agente real.

---

## Decisiones cerradas

### D-01 — Tres agentes separados por componente

**Decision:** tres agentes independientes por componente, cada uno con contexto limpio.

| Agente | Responsabilidad |
|---|---|
| `[comp]_specifier` | Produce la spec detallada a partir del Sprint Contract |
| `[comp]_tester` | Escribe los tests en RED contra la spec |
| `[comp]_executor` | Escribe el codigo hasta que los tests quedan en GREEN |

**Razon:** un tester que hereda el contexto del specifier tiende a escribir tests que confirman lo que el specifier penso, no tests que desafian la implementacion. Contexto limpio produce tests mas adversariales. Alineado con P1 y P4.

---

### D-02 — Modelo Claude por tipo de agente

**Decision:** politica de asignacion fija por tipo de agente, con override declarable en el Sprint Contract de un componente especifico si la complejidad lo justifica.

| Agente | Modelo default | Override permitido |
|---|---|---|
| Orchestrator | Sonnet 4.6 | No |
| [comp]_specifier | Opus 4.7 | No |
| [comp]_tester | Sonnet 4.6 | No |
| [comp]_executor | Sonnet 4.6 | Opus 4.7 si el Sprint Contract lo declara |
| Evaluator | Sonnet 4.6 | No |

**Razon:** el specifier enfrenta la mayor ambiguedad — interpreta requisitos, detecta casos borde, diseña el contrato de comportamiento. Es el unico rol donde el costo de Opus se justifica sistematicamente. El executor puede necesitar Opus en componentes excepcionalmente complejos, pero es la excepcion, no la regla.

---

### D-03 — Branch por componente con repo GitHub obligatorio

**Decision:** repo GitHub publico es prerequisito antes de ejecutar cualquier trabajo. El Orchestrator verifica que el remote esta configurado en la secuencia de inicio (1.11) y falla explicitamente si no lo esta. La URL del repo se solicita al cliente durante el setup del harness.

**Branch strategy:**

| Rama | Proposito | Quien la crea | Cuando se mergea |
|---|---|---|---|
| `main` | Solo codigo con APPROVED | Setup del harness | Solo al recibir APPROVED del Evaluator |
| `sprint/[comp-name]` | Trabajo en curso del componente | Orchestrator al iniciar el ciclo | Al registrar APPROVED — merge ejecutado por el Orchestrator |

**Convencion de commits** — formato `<tipo>(<comp>): <descripcion imperativa corta>`:

| Tipo | Quien lo hace | Cuando |
|---|---|---|
| `spec(<comp>)` | [comp]_specifier | Al producir la spec detallada |
| `test(<comp>)` | [comp]_tester | Al escribir los tests en RED |
| `feat(<comp>)` | [comp]_executor | Al pasar los tests a GREEN |
| `refactor(<comp>)` | [comp]_executor | Fase REFACTOR |
| `eval(<comp>)` | Evaluator | Al firmar la rubrica |
| `chore(<comp>)` | Orchestrator | Actualizaciones de state files y transiciones |
| `fix(<comp>)` | [comp]_executor | Correccion dentro del mismo ciclo |

**Regla de commits:** cada agente commitea exclusivamente su propio trabajo. Sin commits cruzados. Frecuencia: al completar cada artefacto principal, no al finalizar la sesion.

**Razon:** sin repo remoto, una falla de hardware destruye el historial completo, los artefactos commiteados y el punto exacto de reanudacion del harness. GitHub desde el inicio garantiza continuidad en cualquier equipo.
