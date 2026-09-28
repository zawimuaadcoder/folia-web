> Actualización: el proveedor predeterminado es HubSpot. Para conectar el formulario y activar envíos, sigue HUBSPOT.md. Las instrucciones GHL de este documento son alternativas.

# Folia · Marketing Health for Clinics

Landing basada en el frame de Figma `14:181`, con el cuestionario actualizado y recursos locales del diseño.

## Abrir la web

- **Vista previa rápida:** descomprime el ZIP y abre `preview.html` en tu navegador. Conserva la carpeta `public` junto a ese archivo. Esta vista previa funciona sin instalar herramientas y no envía datos.
- **Desarrollo:** instala Node.js 22 o superior, abre una terminal en esta carpeta y ejecuta `npm run dev`. Abre `http://localhost:4173`.
- **Preparar publicación:** `npm run build`.
- **Pruebas de cálculo y API:** `npm test`.
- **Actualizar la vista previa tras editar:** `node scripts/preview.mjs`.

La web no tiene dependencias externas de ejecución. Usa HTML, CSS y JavaScript nativos, una fuente Inter local y funciones de servidor para Netlify.

## Qué está implementado

- Hero con dashboard original, sin tarjetas flotantes.
- Botones negros que cambian a azul de marca; botones azules de la sección oscura que cambian a blanco.
- Flechas que salen por arriba y reaparecen desde abajo en hover/foco.
- Contorno azul giratorio en la etiqueta del hero.
- Navegación a Método, Resultados (testimonio existente) y Calculadora; menú móvil desplegable.
- Layouts adaptados a escritorio, tableta y móvil. Modal a pantalla completa en móvil, scroll interno y botones accesibles.
- Respeto de `prefers-reduced-motion`, navegación con teclado, cierre con Escape y restitución del foco.
- Cinco pasos, rangos con entrada exacta cuando no hay límite superior/inferior, validación, contacto, resultados personalizados, solicitud de auditoría y confirmación.
- Datos conservados en memoria mientras la pestaña está abierta. No se guardan contactos en el navegador.
- Cálculos ilustrativos explicados: sin multiplicadores inventados por zona o equipo, sin promesas de rentabilidad.
- Gestión de errores y reintentos; no muestra una captura exitosa si el servidor de GHL rechaza el envío.

## Publicar en Netlify

1. Crea un repositorio privado en tu cuenta personal de GitHub y sube el contenido de esta carpeta (no el ZIP y no la carpeta `dist`).
2. En Netlify, importa ese repositorio como proyecto.
3. Configuración: comando de build `npm run build`, carpeta publicada `dist`, funciones `netlify/functions`. El archivo `netlify.toml` ya lo declara.
4. Despliega y revisa la web en su URL de Netlify.
5. Conecta tu dominio cuando estés conforme.

**No uses solamente el arrastrar y soltar de `dist`**: esa ruta no despliega las funciones de servidor que necesita la calculadora. Importa el repositorio.

El estado inicial es demostración. Las solicitudes se validan pero no se envían ni guardan. El formulario lo comunica y la confirmación indica explícitamente que es una prueba.

## Conectar GoHighLevel después

La integración está preparada mediante un **Inbound Webhook** de un workflow de GHL. No se ha creado un workflow ni se ha enviado ningún contacto real desde este proyecto.

1. Crea el workflow en tu subcuenta de GHL y añade el trigger **Inbound Webhook**.
2. Obtén su URL. En Netlify, añade `GHL_WEBHOOK_URL` como variable de entorno con alcance de funciones. Nunca la pongas en archivos de `public` ni en GitHub.
3. Mapea el payload a **crear/actualizar contacto**, usando email/teléfono para localizarlo. El webhook por sí solo no garantiza que se cree un contacto; el workflow debe hacerlo.
4. Guarda las respuestas y la proyección en campos personalizados o una nota del contacto. Usa `event_type` para distinguir `analysis`, `audit` y `contact`. Añade etiquetas para análisis desbloqueado y auditoría solicitada.
5. Usa `submission_id` para evitar efectos duplicados en reintentos. El cliente conserva el mismo ID para reintentar un mismo envío; el servidor también lo manda como `X-Idempotency-Key`. La deduplicación duradera corresponde al workflow o almacenamiento que conectes.
6. Para abrir una agenda, configura `GHL_CALENDAR_URL` con la URL HTTPS de tu calendario. Aparecerá después de una solicitud exitosa. No se reserva una cita automáticamente.
7. Configura `PRIVACY_URL`, `LEGAL_URL` y, si corresponde, `COOKIES_URL` con páginas definitivas del titular. Esos textos requieren sus datos reales; esta entrega no los inventa.
8. Cuando todo esté revisado, cambia `FOLIA_MODE` a `live` y vuelve a desplegar. El servidor bloquea los envíos si faltan el webhook o las URLs de privacidad y aviso legal.
9. Haz un envío de prueba controlado y verifica el contacto, campos, etiquetas y auditoría en GHL antes de activar campañas.

Los webhooks entrantes pueden depender del plan o consumo de GHL; compruébalo en tu cuenta. No se incluyen automatizaciones de email/SMS ni se activan comunicaciones por defecto. El consentimiento de marketing es opcional y viaja separado.

### Variables de entorno

| Variable | Uso |
| --- | --- |
| `FOLIA_MODE` | `demo` por defecto; `live` activa el envío real |
| `GHL_WEBHOOK_URL` | URL secreta de destino; solo servidor |
| `GHL_CALENDAR_URL` | Agenda pública opcional |
| `PRIVACY_URL` | Política de privacidad definitiva, HTTPS |
| `LEGAL_URL` | Aviso legal definitivo, HTTPS |
| `COOKIES_URL` | Política de cookies definitiva, HTTPS |
| `SITE_URL` | Opcional: origen canónico exacto, sin barra final |

En local puedes copiar `.env.example` a `.env` y ejecutar `node --env-file=.env scripts/serve.mjs`. No subas `.env` al repositorio.

### Payload enviado a GHL

`event_type`, `submission_id`, `source`, `submitted_at`, `name`, `email`, `phone`, `marketing_consent`, `consent_version`, `note`, `volume_range`, `volume_exact`, `ticket_range`, `ticket_exact`, `has_team`, `zone`, `goal`, `projection`.

El servidor recalcula `projection`; no confía en cifras enviadas por el navegador. Las entradas están validadas, el cuerpo está limitado a 16 KB, hay control de origen y honeypot, y la petición externa tiene un límite de tiempo. Se incluye un límite básico por instancia; para campañas públicas activa un límite persistente o protección antiabuso en Netlify. No se registran cuerpos con datos personales en logs.

## Criterio de la calculadora

Cinco respuestas no permiten una previsión validada del sistema de captación. Por eso el resultado es un **escenario exploratorio**, con hipótesis visibles:

- Volumen: punto medio del rango cerrado; dato concreto en `más de 25`; cero si aún no hay pacientes.
- Ticket: punto medio del rango cerrado; importe exacto en los rangos abiertos.
- Objetivo: aumento ilustrativo del 25 %, redondeado al siguiente paciente. Si parte de cero, escenario de 2 pacientes sin porcentaje de crecimiento.
- Oportunidades adicionales: pacientes adicionales / conversión supuesta del 20 %, redondeando al alza.
- Alternativa de conversión: mismo volumen hipotético de valoraciones, mejor cierre. No se suma a la vía de más valoraciones.
- Ingresos brutos: pacientes × ticket; no descuentan publicidad, honorarios ni costes clínicos.
- Equipo, zona y objetivo contextualizan el texto; no alteran cifras sin evidencia.

## Archivos principales

- `public/index.html`: contenido y estructura.
- `public/styles.css`: tokens, layout responsive y animaciones.
- `public/app.js`: navegación, modal, formularios y estado del recorrido.
- `public/model.js`: preguntas, validación y cálculo compartidos con el servidor.
- `netlify/functions/lead.mjs`: envío a GHL.
- `netlify/functions/config.mjs`: configuración pública sin secretos.
- `public/assets/`: dashboard, tarjetas, logos, iconos, retrato y fuente local.

## Validación y límites de esta entrega

Build y diez pruebas de cálculo, rangos, contacto, modo demo, errores, rechazo de orígenes y payload del webhook ejecutados correctamente. Las pruebas de red usan respuestas simuladas: ninguna conexión real a GHL.

El navegador integrado de este entorno bloquea el servidor y los archivos locales, por lo que **queda pendiente la revisión visual e interacción en navegador real**, especialmente 375 px, 768 px y 1440 px, zoom 200 %, teclado y movimiento reducido. Abre `preview.html` o la URL de Netlify para esa revisión. El código incluye esos layouts y estados, pero no se afirma una verificación visual que no se ha podido ejecutar.

La web no está publicada ni conectada a una cuenta de hosting. El footer contiene avisos de revisión mientras no se configuren los textos definitivos.

Recursos de referencia:
- Figma: https://www.figma.com/design/chiokDN5kWoj8GvcnV89xl/folia?node-id=14-181
- Netlify Functions: https://docs.netlify.com/build/functions/get-started/
- Variables de funciones: https://docs.netlify.com/build/functions/environment-variables/
- GHL Inbound Webhook: https://help.gohighlevel.com/support/solutions/articles/48001237383

Inter se distribuye bajo SIL Open Font License; copia en `public/assets/INTER-LICENSE.txt`. Los recursos gráficos pertenecen al diseño proporcionado por el usuario.
