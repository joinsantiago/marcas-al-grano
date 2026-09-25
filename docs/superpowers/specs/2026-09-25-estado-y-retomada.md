# Estado y retomada — landing Marcas al Grano

**Fecha:** 2026-09-25 · **Último commit:** ver `git log -1` · **Rama:** `main`

Supersede como foto del estado a `2026-09-15-estado-y-retomada.md`. Aquel sigue
valiendo como registro del newsletter y el linktree; lo de aquí es lo que cambió
después.

---

## Lo que cambió en esta tanda

### 1. El libro reemplazó al checklist

La sección que entregaba el *checklist de 12 preguntas* ahora entrega el libro
**«El café ya no compite solo por su sabor»**, de José Arturo López.

- Cambiaron los textos, el botón (*Descargar gratis*), el enlace del pie
  (*Libro*) y el botón del linktree.
- **El id de la sección sigue siendo `#checklist`** en el HTML. No se ve desde
  afuera y renombrarlo obligaba a tocar anclas, CSS y JS por un cambio cosmético.
- **El circuito de Kit no se tocó:** mismo formulario `9889314`, misma
  confirmación por correo.
- **El ebook viejo («De buen café a gran marca», por Web3Forms) sigue donde
  estaba**, por decisión del cliente. La página regala dos cosas.

### 2. Portada real en la sección

`img/libro-portada.jpg` (562 × 900, 76 KB) sale de la **primera página del PDF
del libro**, no del banner promocional: el banner repite el titular y el
subtítulo que ya están escritos al lado.

La clase `.ebook-art.portada` deja que la imagen imponga su proporción. El bloque
original, con la portada dibujada en CSS, sigue sirviendo al ebook viejo.

**Sin `loading="lazy"` a propósito**, por la trampa ya conocida: el bloque entra
con `opacity:0` por la animación de scroll y Chrome pospone la descarga.

### 3. Evento del píxel renombrado

`CompleteRegistration` ahora viaja con
`content_name: 'Libro El cafe ya no compite solo por su sabor'` (antes,
`'Checklist 12 preguntas'`). **La conversión personalizada de Meta hay que
filtrarla por ese texto nuevo.**

### 4. QR de la sección del libro

`qr-libro.svg` (vectorial, para imprenta) y `qr-libro.png` (900 px). Apuntan a
`somosmarcasalgrano.com/#checklist`, con corrección de errores alta para que
aguante manchas o un logo encima. Sin marca de seguimiento: no se puede
distinguir el tráfico del QR del resto.

---

## Estado en el servidor (verificado el 2026-09-25)

| | Estado |
| --- | --- |
| `index.html` con los textos del libro | ✅ Publicado |
| `img/libro-portada.jpg` | ⚠️ **Está el banner, no la portada.** 1081 × 1351 y 401 KB, subido con el nombre cambiado. Hay que reemplazarlo por el del repo (562 × 900, 76 KB) y recargar con Cmd+Shift+R: las imágenes se cachean una semana |
| `links/index.html` | ❌ **Sin subir.** Sigue diciendo «Agenda tu diagnóstico gratuito» y «¿Tu marca está lista para el mercado?» |
| `checklist-12-preguntas.pdf` | ⚠️ **Sigue abierto al público.** Hay que borrarlo de `public_html/` |
| `.htaccess` | ❌ **Sin subir.** `www` sigue sin redirigir al dominio raíz |

---

## 🔴 Lo que está roto ahora mismo: el correo de Kit no llega

**Síntoma:** cuatro pruebas y ningún correo recibido.

**Dónde se cortó, verificado paso a paso:**

| Paso | Estado |
| --- | --- |
| La página manda el correo a Kit | ✅ Muestra «Revisa tu correo» |
| Kit recibe y guarda el suscriptor | ✅ Aparece como *unconfirmed* |
| Kit envía el correo de confirmación | ❌ **Aquí se rompe** |

El fallo **no está en la landing**: el endpoint es correcto en producción y en el
repo, y Kit acepta la petición.

**Qué falta comprobar, en orden:**

1. **Spam y «Promociones».** *Unconfirmed* solo significa «no hizo clic», no «no
   llegó».
2. **Formulario → Settings → Incentive → «Send incentive email» marcado.** Si
   está desmarcado, Kit guarda al suscriptor y no envía nada, que es exactamente
   el síntoma. Es lo más probable, porque justo se estuvo en esa pantalla
   cambiando el PDF.
3. **Si el interruptor está bien: ¿la cuenta está aprobada por Kit?** Una cuenta
   en revisión guarda suscriptores pero no envía correos.

**Dos límites de Kit que condicionan las pruebas:**

- **No se puede reenviar la confirmación** ni confirmar a alguien a mano.
- **El correo no se manda hacia atrás:** encender el interruptor no rescata a
  quien ya se suscribió. Hay que reprobar con correos nuevos (sirve
  `tucorreo+libro2@gmail.com`) y borrar los de prueba.

**Y sigue pendiente lo de siempre en Kit:** cambiar el archivo que se entrega
(*Settings → Incentive → Choose a file*). Hoy sigue entregando el checklist, no
el libro. El PDF pesa 7,6 MB; si Kit lo rechaza, hay que comprimirlo.

---

## Decisiones de esta tanda

- **El PDF del libro y el banner NO van al repo** (`.gitignore`). El repo es
  público: subirlos regalaría el libro sin pedir el correo, que es justo lo que
  la sección existe para evitar. Viven solo en la carpeta local.
- **La portada salió del PDF, no del banner.** El banner es material de anuncio:
  trae titular y subtítulo propios que compiten con los de la sección.
- **El QR apunta a `#checklist`**, el id que ya existe, no a una URL nueva.

---

## Pendientes, por lo que más pesa

1. **Arreglar la entrega de Kit** (arriba). Sin esto, cada suscriptor nuevo se
   pierde: entra a la lista y nunca recibe el libro.
2. **Cambiar el PDF en Kit** al libro.
3. **Subir a Hostinger** `img/libro-portada.jpg` (el bueno), `links/index.html`,
   el `.htaccess`, y **borrar** `checklist-12-preguntas.pdf`.
4. **Actualizar la conversión personalizada de Meta** al `content_name` nuevo.
5. **Probar el recorrido en un teléfono real y dentro del navegador de
   Instagram**, agendando y cancelando. Nunca se ha hecho.
6. **Falta `og:image`**: al compartir por WhatsApp el enlace sale sin imagen.
7. Borrar los suscriptores de prueba de Kit, los viejos y los de hoy.
8. Nombre de usuario para la página de Facebook (hoy el enlace es el ID numérico).
9. Dominio remitente propio en Kit, para que los correos no caigan en spam.

---

## Cómo probar en local sin ensuciar datos

```bash
cd landing
python3 -c "
import pathlib
s = pathlib.Path('index.html').read_text()
pathlib.Path('preview-local.html').write_text(
    s.replace(\"window.META_PIXEL_ID = '2291536674928306';\",
              \"window.META_PIXEL_ID = 'REEMPLAZA_ID_PIXEL';\"))
"
python3 -m http.server 8777    # http://localhost:8777/preview-local.html
```

`preview-local.html` es desechable: borrarlo al terminar y **nunca commitearlo**.
Abrir `index.html` sin esa sustitución dispara eventos reales del píxel. Con
`file://` se rompe el embed de Calendly.
