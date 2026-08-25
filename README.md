# Landing Marcas al Grano®

Landing de captación para **Marcas al Grano®** (branding estratégico para
marcas de café, de la agencia *La Sociedad*). Su único trabajo es conseguir
que un productor, tostador o cafetería **agende un diagnóstico gratuito** de
30 minutos.

Es un sitio **estático**: un solo `index.html` con todo dentro (HTML, CSS y
JS), más imágenes y el PDF del ebook. No hay build, ni dependencias, ni
`npm install`. Se edita el archivo y se sube.

> Este repo es **independiente** del proyecto padre (Panel Agencia). Vive
> dentro de la carpeta `landing/` de ese repo, pero tiene su propio `.git`.
> **Nunca subir la app al hosting de la landing, ni la landing al repo de la
> app.**

---

## Dónde vive

| | |
| --- | --- |
| **Producción** | https://somosmarcasalgrano.com |
| **Repositorio** | https://github.com/joinsantiago/marcas-al-grano (público) |
| **Hosting** | Hostinger, plan Unlimited (vence 2027-08-20) · servidor EEUU, Carolina del Norte |
| **Ruta en el servidor** | `public_html/` (el `index.html` va en la raíz) |
| **GitHub Pages** | **Desactivado** a propósito, para no duplicar el sitio |

### Cómo se despliega hoy

**A mano**, por el Administrador de archivos de Hostinger. Se suben
`index.html`, `img/`, `ebook-marcas-al-grano.pdf` y `.htaccess` a
`public_html/`.

El despliegue automático por Git **está pendiente** (ver
[Pendientes](#pendientes)). Mientras tanto, cada cambio requiere subir el
archivo a mano, aunque el repo esté actualizado.

### Cómo probarla en local

```bash
cd landing
python3 -m http.server 8777
# abrir http://localhost:8777
```

> Ojo: al abrirla en local, el píxel de Meta **dispara eventos reales**.
> Para probar sin ensuciar datos, vaciar `META_PIXEL_ID` en la copia local.
> Abrirla con doble clic (`file://`) **no sirve**: rompe el embed de Calendly.

---

## Integraciones

Toda la configuración vive en el objeto `CONFIG`, al inicio del `<script>`
al final del `index.html`. Es el único lugar que hay que tocar para cambiar
enlaces o claves.

| Servicio | Valor | Notas |
| --- | --- | --- |
| **Píxel de Meta** | `2291536674928306` | En el `<head>`, no en `CONFIG`. Verificado disparando eventos en producción |
| **Calendly** | `calendly.com/jlopeztu/la-sociedad-agencia` | Evento "Sesión Estratégica para Marcas de Café", 30 min. **Plan gratuito** |
| **Video VSL** | YouTube `dDSLKoJLxeU` | 2 min 10 s, **no listado** |
| **Testimonios** | `f9avuhQzspY` (Camoro, 16:9) · `mdejUZ9PRiA` (La Marquesa, Short 9:16) | No listados |
| **Formularios** | Web3Forms, key `e19ce545-b94c-485d-aecc-ee57c10ff14a` | Los leads llegan a **jlopeztu@gmail.com** |
| **WhatsApp de respaldo** | `573160597375` | Solo aparece si falla el envío del formulario |

### Web3Forms: límites que importan

- **250 envíos/mes** en el plan gratuito, **compartidos** entre el formulario
  de diagnóstico y el del ebook. Al superarlos, **deja de aceptar envíos
  hasta el mes siguiente** y los leads se pierden en silencio.
- **Los envíos de más de 30 días desaparecen de su panel.** El correo de
  Jose Arturo es el único registro permanente: conviene una etiqueta en
  Gmail que no se borre.
- La access key es **pública por diseño** (viaja en el HTML). No es un
  secreto, pero **nunca poner ahí tokens que sí lo sean**: el repo es público.
- Web3Forms **solo acepta peticiones desde un navegador**. Probarlo con
  `curl` desde un servidor devuelve "method is not allowed" — no es un fallo
  de la clave.

### Calendly: el plan gratuito y su promoción

En el plan gratuito, Calendly le muestra al invitado **una promoción suya
justo después de reservar**. Para evitar que el prospecto termine viendo
publicidad de Calendly dentro de la landing, `onCalendlyScheduled()` retira
el iframe apenas llega el evento `calendly.event_scheduled` y lo reemplaza
por una confirmación propia.

Quedan dos cosas del plan gratuito que **no se pueden quitar sin pagar**:
el cintillo "Desarrollado por Calendly" en la esquina del widget, y el fondo
blanco (los parámetros de color se ignoran fuera de los planes de pago).

Dos bugs ya resueltos, por si reaparecen:
1. Faltaba cargar `widget.css`; sin ella el spinner queda en alto 0.
2. Calendly no le pone su clase al contenedor que le pasamos, así que el
   alto del iframe lo fija nuestro CSS (`#cal-embed iframe`).

---

## Eventos del píxel

Los envía `metaTrack()` / `metaTrackCustom()`, con `metaIdentity()` para
advanced matching (correo y teléfono hasheados por Meta en el navegador).

| Evento | Cuándo | Tipo |
| --- | --- | --- |
| `PageView` | Al cargar | Estándar |
| `VerVideo` | Play al VSL | Personalizado |
| `VerTestimonio` | Play a un testimonio | Personalizado |
| `AbreAgenda` | Al abrir el modal (incluye qué programa) | Personalizado |
| `FormPaso2` | Al pasar al paso 2 del formulario | Personalizado |
| **`Lead`** | Envío con presupuesto **calificado** | Estándar ← *el que importa* |
| `LeadNoCalificado` | Envío con "Menos de USD $1.000" | Personalizado |
| `Schedule` | Calendly confirma la reserva | Estándar |
| `CompleteRegistration` | Descarga del ebook | Estándar |
| `Contact` | Clic en el WhatsApp de respaldo | Estándar |

**Por qué `Lead` y `LeadNoCalificado` están separados:** si se optimiza por
`Lead` a secas, Meta trae gente sin presupuesto porque es más barata. Con la
separación se optimiza por el bueno y se puede excluir al otro.

---

## Estructura de la página

```
hero → video (VSL) → problema → cita → programas → metodología
     → casos → testimonios → agenda → ebook → FAQ → footer
```

Decisiones que no son obvias al leer el HTML:

- **El ebook va DESPUÉS de agendar.** Antes estaba justo encima del
  formulario, preguntando "¿Todavía no es momento de una llamada?" — le
  ofrecía una salida fácil a quien ya venía convencido.
- **No hay sección de precios.** Se eliminó "Inversión" por decisión del
  cliente. Los precios solo se insinúan en los rangos del formulario.
- **El formulario vive en un modal** (`#agendaModal`), no en la página. Cada
  botón "Es para mí" de un programa lo abre **con ese programa ya
  seleccionado**. Todo lo que apunte a `#agenda` abre el modal.
- **Los testimonios se arman desde `CONFIG.testimonios`.** Si la lista queda
  vacía, la sección **se oculta sola** en vez de dejar un hueco.
- **El logo es un PNG negro usado como máscara CSS**, no como imagen: así
  toma el color de la marca y cambia en hover.

### Los 4 programas

| Programa | Duración | Precio | Qué es |
| --- | --- | --- | --- |
| Estrategia al Grano® | Entrega única | USD 1.200 | La hoja de ruta, sin acompañamiento |
| Marca Base® | 30 días | USD 1.500 | Marca desde cero |
| Escala tu Marca® | 45 a 90 días | USD 2.300 | Evolución de una marca existente |
| Dirección Estratégica® | 90 días | USD 1.000/mes | Acompañamiento con comité mensual |

> **Cuidado al retomar:** *Estrategia al Grano* y *Dirección Estratégica*
> intercambiaron su significado respecto a la versión vieja de la landing.
> Hoy **Dirección Estratégica es el acompañamiento** y *Estrategia al Grano*
> la entrega única. Cualquier texto viejo dice lo contrario.
>
> *Formación al Grano®* fue **eliminado**.

### Calificación por presupuesto

Rangos en USD. Quien elige **"Menos de USD $1.000"** no ve el calendario:
recibe un mensaje de descarte y el ebook. La lógica es
`data.inversion.startsWith('Menos de')`.

---

## Trampas ya pisadas

Cosas que costaron encontrar y que conviene no repetir:

- **`loading="lazy"` en la portada del video**: el bloque entra con
  `opacity:0` por la animación de scroll, y Chrome posponía la descarga
  indefinidamente. Quitado a propósito.
- **Los Shorts de YouTube tienen miniatura vertical propia**
  (`oardefault.jpg`, 720×1280). Usar la estándar en un marco 9/16 recorta la
  cara. El código prueba las variantes en orden hasta que una carga.
- **`window.track` chocaba** con `const track = [...]` (los pasos del
  formulario). Por eso los helpers se llaman `metaTrack*`.
- **El botón del menú móvil** se posicionaba con un cálculo que asumía 5
  enlaces. Ahora mide la lista real y la pasa por `--links-h`.
- **Los `scrollIntoView` del formulario** mueven el `.modal-card`, no la
  página de atrás (`subirAlInicioDelFormulario()`).

---

## Pendientes

Ordenados por lo que más pesa sobre la agenda.

### Antes de invertir en anuncios

1. **Probar el recorrido completo en un teléfono real.** Es lo único del
   funnel que nunca se verificó en condiciones reales, y de ahí vendrá la
   mayoría del tráfico de Meta. Hay que **agendar de verdad y cancelar**:
   eso confirma de un golpe el prefill de Calendly, el evento `Schedule` y
   el comportamiento del modal en móvil.
2. **Verificación de dominio en Meta.** Estado **incierto**: Meta no pidió
   código, lo que puede significar que lo verificó solo (ya recibe eventos
   con el dominio identificado) o que quedó pendiente. Hay que mirar si
   aparece con check verde en *Configuración del negocio → Dominios*. Sin
   ella se pierde atribución en iOS.

### Mejoras claras

3. **Falta `og:image`.** Al compartir el enlace por WhatsApp sale sin
   imagen de vista previa, lo que cuesta clics en tráfico pagado. Se puede
   generar con el logo sobre el fondo oscuro de la marca (1200×630).
4. **Despliegue por Git en Hostinger.** Bloqueado: el panel muestra
   *"Deshabilitado porque estás suplantando la cuenta de usuario"*. Hay que
   entrar como **titular** de la cuenta de Hostinger para conectar GitHub.
   Mientras tanto, todo despliegue es manual.
5. **`www` y el dominio raíz sirven por separado**, sin redirigir uno al
   otro. Es cosmético (el `canonical` ya resuelve el SEO y el píxel no se
   ve afectado), pero ensucia los informes. El `.htaccess` del repo ya lo
   corrige — **solo hay que subirlo**, cosa que aún no se hizo.

### Del análisis de conversión

Auditoría completa: https://claude.ai/code/artifact/3f260ef4-6f76-4dff-b49f-6411b0741133

6. **Los casos muestran el trabajo, no el resultado.** Seis fichas sin una
   sola métrica de qué pasó después. Falta una línea de resultado por caso.
7. **No se sabe quién dirige la consultoría.** La página habla en "nosotros"
   y nunca presenta a Jose Arturo López. Para un ticket de USD 1.200–2.300,
   el criterio de quien la dirige *es* el producto.
8. **Seis casos diluyen.** Tres con resultado convencen más.
9. **Nada indica que la agenda sea limitada.** Lo gratuito e ilimitado se
   pospone. Solo si es cierto.

### Preguntas abiertas

- **¿"Camoro Café" y el caso "Camoro Origen" son la misma marca?** Si sí,
  conviene unificar el nombre en toda la página.
- **Dos números de WhatsApp distintos**: la landing usa `316 059 7375`; el
  PDF del ebook cierra con `316 495 3842`. El cliente decidió **dejarlo
  así**, pero el del PDF solo se cambia reexportando el ebook.
- **El ebook está firmado por "la sociedad." y "MAG®//2026"**, no por Marcas
  al Grano. Confirmado como intencional.

---

## Decisiones tomadas (y por qué)

- **El video no se aloja aquí.** El VSL pesa 155 MB: Git rechaza archivos de
  más de 100 MB y los términos de GitHub Pages prohíben usarlo como CDN de
  video. Va en YouTube no listado. Lo mismo aplica a Hostinger.
- **Se quitaron tres testimonios inventados** (Carlos Mendoza, María López,
  Andrés Ruiz) que no correspondían a ningún caso real de la página, junto
  con la afirmación "subir precios un 40%" — una promesa de resultado
  económico que Meta restringe en anuncios.
- **`.htaccess` cachea imágenes y el PDF, pero no el `index.html`**, para
  que cada despliegue se vea de inmediato.
- **El titular va en mayúscula y minúscula**, no en mayúsculas sostenidas:
  en Playfair a ese tamaño se lee peor y pelea con el logo.
