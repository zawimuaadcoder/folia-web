# Conexión con HubSpot

Esta versión sustituye el destino predeterminado GHL por el formulario de HubSpot facilitado por el propietario. Conserva la web y la calculadora. No requiere insertar el script de HubSpot ni una clave API.

## Publicación

1. Sustituir en GitHub los archivos del proyecto por los de este ZIP, conservando la estructura de carpetas. Netlify construirá la nueva versión.
2. En Netlify, configurar las variables de entorno para Functions en producción:
   - CRM_PROVIDER = hubspot
   - FOLIA_MODE = live
   - PRIVACY_URL = URL HTTPS real de la política de privacidad publicada
   - LEGAL_URL = URL HTTPS real del aviso legal publicado
3. Volver a desplegar. Los identificadores de cuenta 149430727 y formulario 97ba49d3-ce4d-4b1a-a503-6674f862e605 ya están incluidos; pueden sobreescribirse con HUBSPOT_PORTAL_ID y HUBSPOT_FORM_ID.
4. Probar con datos propios y comprobar el envío en HubSpot. Verificar los tres recorridos: desbloquear análisis, solicitar auditoría y contacto directo.

## Formulario requerido

El formulario publicado debe contener las propiedades de contacto firstname (Nombre), email y phone (Teléfono). El nombre completo se guarda íntegro en firstname para no adivinar apellidos. Si hay otros campos obligatorios o requisitos de consentimiento del formulario, hace falta adaptar el envío antes de activarlo. No se omiten ni se falsifican consentimientos. El código de inserción recibido identifica el formulario, pero no permite verificar su configuración interna.

El modo live conserva la comprobación de URLs legales existente. No usar URLs de ejemplo. Mientras falten estas variables, el envío no se activará correctamente. No se han creado textos legales con datos ficticios.

## Respuestas y resultados (opcional)

De forma predeterminada se guardan nombre, email y teléfono. El contexto de cada envío indica análisis, auditoría o contacto; no se crea automáticamente una oportunidad comercial.

Para guardar el diagnóstico, añadir al formulario publicado una propiedad de texto multilínea (por ejemplo, la propiedad Mensaje si su nombre interno es message). Configurar HUBSPOT_SUMMARY_FIELD con ese nombre interno y desplegar otra vez. El resumen JSON incluirá respuestas, proyección calculada por el servidor, motivo de contacto, nota e identificador de envío. No se envían campos que aún no se hayan añadido al formulario. El valor actual de la propiedad puede sobrescribirse con envíos posteriores; revisar también el historial de envíos.

La casilla opcional de marketing queda oculta con HubSpot hasta configurar su tipo de suscripción y el consentimiento correspondiente. No se activan newsletters ni automatizaciones de email.

## Validación y límites

Pruebas automáticas con respuestas de HubSpot simuladas. Pendiente prueba real desde Netlify y comprobación dentro del CRM. No se ha accedido a la cuenta, enviado contactos, actualizado GitHub ni desplegado desde este entorno.

Forms API puede rechazar formularios con CAPTCHA o campos/consentimientos que no coincidan con este payload. Si ocurre, revisar la configuración y adaptar la integración; no desactivar controles como solución automática. Los rechazos nunca se presentan como envíos correctos. Los reintentos tras un timeout pueden producir entradas de formulario duplicadas: HubSpot Forms no garantiza idempotencia con la cabecera enviada.

Documentación: https://developers.hubspot.com/changelog/announcing-forms-submission-rate-limits
https://developers.hubspot.com/changelog/reminder-validation-change-to-the-forms-api-submission-endpoints-0

El apartado GHL del README corresponde al proveedor anterior; esta guía prevalece para HubSpot. GHL sigue disponible únicamente al configurar CRM_PROVIDER=ghl.
