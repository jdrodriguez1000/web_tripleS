# Alcance del Proyecto — Pagina Web Sabbia Solutions & Services (Triple S)

**Ultima actualizacion:** 2026-05-22
**Estado:** Definido — FASE 0 cerrada

---

## Proposito

Define con precision que se construye y que NO se construye.
Cada item tiene su decision de referencia en `decisiones.md`.

---

## Dentro del alcance

### Pagina web publica (landing page)

| Item | Detalle | Decision |
|------|---------|----------|
| Hero | Tagline + descripcion breve + CTA "Solicitar reunion" | D-05, D-06 |
| Casos de exito | Seccion con metricas reales de clientes (reduccion agotados, ahorro inventario, etc.) | D-05 |
| Como funciona | Pasos del proceso: datos entrada → IA → pronostico → accion del cliente | D-05 |
| Que gana la empresa | Beneficios concretos expresados en lenguaje de negocio | D-05, D-07 |
| Precios | Planes o tiers de la plataforma | D-05 |
| Integraciones ERP | SAP, Oracle y otros ERPs del mercado | D-05 |
| CTA / Formulario | Formulario: nombre, empresa, correo, telefono (opcional), mensaje (opcional) | D-05, D-06 |
| Footer | Informacion empresa, links, redes sociales | D-05 |
| Idiomas | Espanol e Ingles con toggle en header. Default: Espanol | D-03 |
| Identidad visual | Azul `#0F1F3D`, cian `#00C2FF`, Inter, hero oscuro | D-01 |
| Diseno responsive | Mobile, tablet y desktop | D-02 |

### Backend / Datos

| Item | Detalle | Decision |
|------|---------|----------|
| API de contacto | Endpoint que recibe el formulario y guarda en PostgreSQL | D-04 |
| Tabla leads | PostgreSQL local con schema definido en D-04 | D-04 |

### Contenido editable sin codigo

| Item | Detalle | Decision |
|------|---------|----------|
| `content/pricing.json` | Planes, precios y features — editable sin tocar componentes | D-09 |
| `content/cases.json` | Casos de exito ficticios: empresas colombianas en Medellin, Bucaramanga, Barranquilla | D-08, D-09 |

---

## Fuera del alcance (v1)

| Item | Razon |
|------|-------|
| Login / autenticacion de usuarios | Es la plataforma B2B, no la pagina publica |
| La plataforma de pronostico de demanda | Producto separado — la web solo la presenta |
| Panel de administracion para gestionar leads | Se gestiona directamente en BD por ahora |
| Integracion con CRM (HubSpot, Salesforce, etc.) | Diferido para v2 si el volumen lo justifica |
| Blog o gestion de contenido (CMS) | No solicitado en v1 |
| E-commerce / pagos en linea | El ciclo de venta es consultivo, no transaccional |
| Chat en vivo o chatbot | No solicitado |
| Analytics dashboard propio | Se puede agregar Google Analytics o similar externamente |
| Aplicacion movil | Solo web |
| Tercer idioma u otros idiomas | Solo ES y EN en v1 |

---

## Criterios de exito

Una pagina esta terminada y correcta cuando:

1. Las 8 secciones estan implementadas en el orden definido (D-05)
2. El toggle ES/EN funciona y todos los textos tienen traduccion completa (D-03)
3. El formulario guarda correctamente un lead en la tabla PostgreSQL (D-04)
4. El diseno es fiel a la identidad visual definida (D-01)
5. La pagina es completamente responsive (mobile, tablet, desktop)
6. Todos los CTAs llevan al formulario de contacto
7. La pagina pasa Lighthouse con score >= 90 en Performance y >= 95 en SEO
