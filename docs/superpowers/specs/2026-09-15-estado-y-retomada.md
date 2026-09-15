# Estado y retomada — landing Marcas al Grano

**Fecha:** 2026-09-15 · **Último commit:** `c7cb02f` · **Repo y GitHub sincronizados**

Este documento reemplaza como foto del estado a los dos specs anteriores
(`2026-09-06-newsletter-checklist-kit-design.md` y su plan), que siguen siendo
válidos como registro de **por qué** se tomó cada decisión.

---

## Qué hay publicado ahora mismo

Verificado contra el servidor el 2026-09-15, no asumido:

| | Estado |
| --- | --- |
| `somosmarcasalgrano.com` | ✅ 200 · 84.897 bytes · **con la sección del checklist** |
| `somosmarcasalgrano.com/links` | ✅ 200 · 12.280 bytes · **el linktree** |
| Píxel en ambas | ✅ `2291536674928306` |
| Endpoint de Kit en producción | ✅ `app.kit.com/forms/9889314/subscriptions` |
| Enlaces de redes del linktree | ✅ los cuatro correctos |
| `README.md` y `docs/` en el servidor | ✅ 404 — no se publicó nada interno |

---

## Lo que se construyó en esta tanda

### 1. Newsletter: el checklist por correo con Kit

Sección `#checklist` entre `#agenda` y `#ebook`. Captura el correo, lo manda al
formulario `9889314` de Kit y Kit entrega el PDF tras el doble opt-in.

- Kit acepta el POST cross-origin (`status 200`, `type cors`), así que la landing
  **lee la respuesta** y distingue el fallo del éxito. El campo debe llamarse
  `email_address`.
- **Es un solo correo**: el PDF llega en la redirección posterior al clic de
  confirmación, no en un segundo correo.
- Dispara `CompleteRegistration` con `content_name: 'Checklist 12 preguntas'`.
  **Nunca `Lead`.**
- El ebook no se tocó: sigue en Web3Forms.

### 2. Linktree en `/links`

Página aparte para el bio de Instagram. Agendar primero, checklist segundo,
redes como iconos. Cada botón dispara `ClicBio` con el nombre del botón.

El detalle completo de ambas piezas —y el porqué de cada decisión— está en el
README, que se actualizó con las dos.

---

## Pendientes, por lo que más pesa

### 1. Borrar el PDF del servidor ⚠️

`somosmarcasalgrano.com/checklist-12-preguntas.pdf` responde **200**: el archivo
se subió a `public_html/` por error. Cualquiera puede abrirlo directo y llevarse
el checklist **sin dejar el correo**, que es justo lo que la sección existe para
evitar.

**Qué hacer:** borrarlo desde el Administrador de archivos de Hostinger. El PDF
lo entrega Kit; no hace falta en el servidor.

### 2. Probar el recorrido dentro del navegador de Instagram

El linktree vive en el bio, y los enlaces del bio **no abren en Safari ni en
Chrome, sino dentro de la app**. Ahí el botón de agendar abre el modal con el
embed de Calendly, que ya dio problemas en condiciones normales.

**Nunca se ha probado.** Si el calendario no carga bien, la salida es mandar ese
botón directo a Calendly en vez de al modal: es un cambio de una línea.

### 3. Confirmar que los perfiles de Instagram y TikTok son los correctos

`instagram.com/marcasalgrano` y `tiktok.com/@marcasalgrano` responden, pero ambas
redes muestran un muro a quien consulta sin sesión, así que **no se pudo
verificar que sean los perfiles de la marca**. Se confirma abriéndolos desde el
teléfono.

### 4. Borrar los suscriptores de prueba de Kit

`prueba-cors@` y `prueba-landing@somosmarcasalgrano.com`, usados para verificar
el circuito.

### 5. Comprobar el evento en Meta y crear la conversión personalizada

Confirmar en el Administrador de eventos que aparece `CompleteRegistration` con
`content_name: Checklist 12 preguntas` y que **no** se dispara ningún `Lead`.
Luego crear la conversión personalizada filtrada por ese `content_name`, para
separar checklist y ebook en los informes.

### 6. Subir el `.htaccess`

`www.somosmarcasalgrano.com` sigue respondiendo 200 sin redirigir al dominio
raíz, así que la regla del `.htaccess` del repo **no está actuando**. Lleva meses
pendiente. Es cosmético pero ensucia los informes.

### 7. Nombre de usuario para la página de Facebook

Hoy el enlace es el ID numérico `61573679933526` porque la página no tiene
usuario asignado. Con uno queda `facebook.com/<usuario>`: más corto, buscable, y
es cambiar una línea en `links/index.html`.

### 8. Dominio remitente propio en Kit

Los correos salen desde la infraestructura de Kit. Configurar
`somosmarcasalgrano.com` con sus registros DNS en Hostinger mejora que no caigan
en spam.

---

## Cosas que conviene no olvidar al retomar

- **El despliegue es manual.** Hacer `git push` **no publica nada**. Hay que
  subir los archivos por el Administrador de archivos de Hostinger. El
  despliegue por Git sigue bloqueado: hay que entrar como **titular** de la
  cuenta, no con acceso delegado.
- **Qué se sube y qué no:** solo `index.html` a la raíz y `links/index.html`
  dentro de `public_html/links/`. **Nunca** el PDF del checklist, ni `docs/`, ni
  el `README.md`.
- **El gestor de Hostinger no acepta carpetas**, solo archivos sueltos. La
  carpeta se crea en el servidor y se entra en ella antes de subir.
- **Los dos `index.html` se llaman igual a propósito.** Se distinguen por el
  peso: **83 KB** es la landing, **12 KB** el linktree. El de 12 KB no debe
  reemplazar nunca nada.
- **Abrir la landing en local dispara eventos reales del píxel.** Antes de
  probar, poner `META_PIXEL_ID` en `REEMPLAZA_ID_PIXEL`, y restaurarlo antes de
  cualquier commit.
- **La prueba social dice 12 marcas.** El cliente cree que son más y prefirió
  dejarlo así.

---

## Respaldo

En `~/Downloads/backup-marcas-al-grano-2026-09-15/`:

| Archivo | Qué es |
| --- | --- |
| `marcas-al-grano-repo-completo.bundle` | El repositorio entero con toda su historia. Verificado: *"records a complete history"*. Se restaura con `git clone <archivo>.bundle carpeta-nueva` |
| `landing-carpeta-completa.zip` | La carpeta de trabajo sin `.git`: 25 archivos, incluidos los dos PDF y las imágenes |
| `produccion-antes-de-tocar/index.html` | Lo que servía el dominio ese día (83 KB) |
| `produccion-antes-de-tocar/links-index.html` | Lo que servía `/links` ese día (12 KB) |

Los dos últimos son el punto de retorno: si una subida rompe el sitio, se vuelven
a subir tal cual y queda como estaba.

**El respaldo vive en `~/Downloads`, en el mismo disco que el proyecto.** Eso lo
protege de un error al desplegar, pero no de perder el equipo. Para eso está
GitHub, que tiene todo el código —aunque no los PDF de la carpeta de trabajo ni
la copia de producción.
