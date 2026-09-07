# Newsletter: entrega del checklist por correo con Kit

**Fecha:** 2026-09-06 · **Estado:** diseño aprobado, pendiente de implementar

## Qué se quiere

Que un visitante deje su correo en la landing, **reciba en su bandeja** el PDF
*Checklist: 12 preguntas antes de lanzar o renovar tu marca de café* (16
páginas), y **quede en una lista** a la que se le puedan enviar números
posteriores.

Hoy nada de esto existe. El formulario del ebook (`index.html:1631`) manda el
correo del visitante **a Jose Arturo** vía Web3Forms y le muestra un enlace de
descarga en la misma página. El suscriptor no recibe ningún correo, y no queda
lista a la que escribirle después.

## Qué NO entra

- **El ebook no se toca.** Sigue con Web3Forms y su enlace en página. No
  necesita entrega por correo, y dejarlo fuera libera el único formulario que
  da Kit en el plan gratuito.
- **No se migran los suscriptores viejos del ebook.** Nunca se pidió permiso
  para enviarles un newsletter; meterlos en la lista es lo que genera quejas de
  spam y quema el dominio.
- **No se configura dominio remitente propio** en esta entrega. Ver *Pendientes*.

## Decisiones y por qué

### Kit (ex ConvertKit), plan gratuito

| | Kit | Brevo | MailerLite |
| --- | --- | --- | --- |
| Suscriptores gratis | **10.000** | 100.000 almacenados | 250 |
| Envíos | Ilimitados | 300/día | 2.500/mes |
| Formularios | **1** | Ilimitados | Varios |
| Automatizaciones | 1 | Tope 2.000 contactos **en total** | Incluidas |

Se elige **Kit**:

- El techo de 10.000 suscriptores no se toca en años. Los 250 de MailerLite se
  llenan en semanas con tráfico pagado de Meta.
- Trae la entrega del imán de fábrica: el correo de confirmación se configura
  **por formulario** (*Settings → Confirmation email*) y en *After confirming
  redirect to* se elige **Download** con el archivo. No consume la única
  automatización del plan.
- Publicar un número es escribir un *broadcast* y enviarlo. No requiere tocar
  código ni depender de Santiago.

**Se descarta Brevo** pese a tener el panel en español: su automatización
gratuita tiene tope de **2.000 contactos totales**, y al llegar ahí los correos
de bienvenida **dejan de salir en silencio** — la misma trampa que este README
ya documentó con la cuota de Web3Forms. Sumado al tope de 300 correos/día, no
sirve para newsletter.

**Se descarta MailerLite** por los 250 suscriptores (recorte de junio de 2026).

**Se descarta el autoresponder de Web3Forms**: es función de pago, y aun
pagando no adjunta archivos.

**Se descarta n8n**, que ya corre en la máquina de Santiago vía PM2: si el
equipo está apagado, el suscriptor no recibe nada. No es infraestructura de
producción.

### La sección va después de agendar

```
… agenda → checklist (NUEVO) → ebook → FAQ → footer
```

El README ya registra por qué el ebook se movió debajo del formulario: encima
le daba una salida fácil a quien venía convencido. El checklist tiene ese mismo
riesgo, y mayor, porque es más apetecible.

Va **por delante del ebook** porque es la pieza más fuerte: el PDF hace que el
lector se ponga una nota sobre 12 y cierra con *"si descubriste que todavía
existen preguntas sin resolver, conversemos"*. Es el único imán que devuelve
gente al calendario.

### No hay descarga inmediata en la página

Kit entrega el PDF **después** de que el suscriptor confirme su correo (doble
opt-in). Si además dejáramos un enlace de descarga en la página, nadie
confirmaría y no habría lista — que es justamente el objetivo.

Se acepta el costo: se pierde a quien nunca confirma. A cambio la lista queda
formada por gente que abre correos, que es lo que sostiene un newsletter.

### El píxel: evento `CompleteRegistration`, nunca `Lead`

Se dispara `CompleteRegistration` con `content_name: 'Checklist 12 preguntas'`
—el mismo evento estándar que ya usa el ebook, distinguido por el nombre— más
`metaIdentity(correo)` para advanced matching. Sobre ese `content_name` se crea
en el Administrador de Meta una **conversión personalizada**.

**No se dispara `Lead`.** El README es explícito: `Lead` está reservado a quien
declara presupuesto calificado, y contaminarlo hace que Meta traiga gente sin
dinero porque es más barata. Suscriptores de un PDF gratuito ahí arruinarían la
optimización de los anuncios.

### El PDF lo aloja Kit

El archivo se sube a Kit y desde ahí se sirve. No se sube a Hostinger: así el
enlace no es adivinable y no se puede saltar el formulario.

Se renombra a `checklist-12-preguntas.pdf` — el nombre actual tiene espacios y
una tilde, y eso rompe URLs. Queda una copia en el repo para control de
versiones.

**El bloqueo es blando:** el repo de la landing es público, así que quien
busque el archivo en GitHub lo encuentra. Es la misma situación del ebook y se
acepta igual.

## Cambios en `index.html`

### CONFIG

Se añade una clave, junto a las que ya existen:

```js
// URL de acción del formulario de Kit que entrega el checklist.
// Se saca del embed del formulario en app.kit.com. Vacío = no envía nada.
checklistEndpoint: 'https://app.kit.com/forms/XXXXXXX/subscriptions',
```

No se añade `checklistFile`: el PDF lo entrega Kit, la landing nunca enlaza al
archivo.

### Sección `#checklist`

Se inserta entre `#agenda` y `#ebook`, reutilizando el patrón visual de
`.ebook` (tarjeta ilustrada + formulario al lado). No se inventa lenguaje nuevo
ni CSS nuevo salvo lo mínimo para reusar `.ebook-art` con otro texto.

Copia propuesta:

- Eyebrow: `Autodiagnóstico gratuito`
- H2: `¿Tu marca está lista para salir al mercado?`
- Lead: `Responde 12 preguntas y ponle una nota sobre 12 a tu marca. En diez
  minutos vas a saber si es momento de diseñar, de corregir, o de detenerte
  antes de gastar en empaques que no funcionan.`
- Botón: `Recibir el checklist`
- Nota bajo el formulario: `16 páginas · PDF · Te llega por correo`
- Aviso de suscripción, obligatorio por honestidad y para reducir quejas de
  spam: `Al suscribirte quedas en la lista de Marcas al Grano. Puedes darte de
  baja cuando quieras.`

Estados del formulario:

| Estado | Mensaje |
| --- | --- |
| Correo inválido | `Escribe un correo válido.` (en rojo, como el ebook) |
| Enviando | El botón pasa a `Enviando…` y se deshabilita |
| Éxito | `Revisa tu correo: te enviamos un enlace para confirmar. Al confirmar se descarga tu checklist.` |
| Fallo de red | `No pudimos suscribirte. Escríbenos por WhatsApp y te lo enviamos.` con el `whatsapp` de respaldo que ya existe en CONFIG |

### Menú

**No se añade al menú de cabecera**: competiría con el botón "Agenda tu
diagnóstico", que es el trabajo de la página. **Sí se añade al pie**, junto a
Programas / Metodología / Casos / FAQ.

El pie es una lista suelta y no interviene en el menú móvil: el cálculo de
`--links-h` mide la lista de la cabecera, que no cambia. Añadir el enlace al
pie no toca nada de eso.

## Riesgo técnico: CORS

**El punto que hay que verificar antes de construir el resto.** La landing es
estática y envía con `fetch()` para conservar su propio diseño y sus estados de
éxito. No está confirmado que el endpoint de formularios de Kit acepte una
petición cross-origin desde `somosmarcasalgrano.com`.

Orden de intentos:

1. **`fetch()` normal** al endpoint del formulario, leyendo la respuesta.
2. **`fetch()` con `mode: 'no-cors'`**: la petición sale y el suscriptor queda
   registrado, pero no se puede leer la respuesta. Se muestra éxito de forma
   optimista. Es exactamente lo que ya hace el formulario del ebook, que ignora
   los errores con `catch(_){}`.
3. **Embed JS propio de Kit**, estilado para encajar. Se pierde control del
   marcado y del mensaje de éxito, pero funciona seguro.

Se resuelve con una prueba de diez minutos al empezar, no discutiéndolo.

## Verificación

Ni el correo de Kit ni el píxel se comprueban en local: Kit solo dispara en
sitios en producción, y el píxel en local ensucia datos reales. La prueba es en
producción, con un correo real:

1. Suscribirse desde la sección nueva → llega el correo de confirmación.
2. Confirmar → el clic lleva a la descarga del PDF, y el PDF abre.
3. El suscriptor aparece en el panel de Kit.
4. En el Administrador de eventos de Meta aparece `CompleteRegistration` con
   `content_name: 'Checklist 12 preguntas'`.
5. **Repetir todo desde un teléfono real.** Es el pendiente número 1 del README
   y aquí se cubre de paso.
6. Comprobar que el formulario del ebook sigue funcionando igual.

## Despliegue

Manual por el Administrador de archivos de Hostinger, como todo en este repo
mientras el despliegue por Git siga bloqueado. **Solo se sube `index.html`**:
el PDF no va al servidor.

## Reparto de trabajo

**Jose Arturo / Santiago (en Kit y Meta):**

1. Crear el formulario en Kit y poner su confirmación en modo **Download**.
2. Subir `checklist-12-preguntas.pdf` a Kit.
3. Redactar en español el correo de confirmación y el de entrega.
4. Entregar la **URL de acción del formulario** para ponerla en CONFIG.
5. Crear la conversión personalizada en Meta filtrada por el `content_name`.

**Implementación en el repo:**

1. Renombrar el PDF y versionarlo.
2. Escribir la sección `#checklist`, su formulario, sus estados y el evento.
3. Añadir el enlace del pie.
4. Actualizar el README: integraciones, tabla de eventos, mapa de secciones y
   pendientes.

## Pendientes que deja abiertos

- **Dominio remitente propio.** Kit envía desde su infraestructura. Configurar
  `somosmarcasalgrano.com` con sus registros DNS en Hostinger mejora la
  entregabilidad. Se deja para después de que el circuito funcione.
- **Marca de Kit al pie de cada correo.** No se quita sin pagar.
- **Kit gratuito da un solo formulario y una sola automatización.** Llevar
  también el ebook a Kit exige plan de pago.
- **El panel de Kit está en inglés.** Los correos al suscriptor van en español,
  pero quien envíe cada número navega una interfaz en inglés.
