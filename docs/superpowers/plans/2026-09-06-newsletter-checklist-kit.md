# Sección checklist + newsletter con Kit — Plan de implementación

> **Para agentes:** SUB-SKILL OBLIGATORIA: usa superpowers:subagent-driven-development (recomendada) o superpowers:executing-plans para ejecutar este plan tarea por tarea. Los pasos usan casillas (`- [ ]`) para llevar el control.

**Objetivo:** Añadir a la landing una sección que capte correos, entregue el *Checklist: 12 preguntas* por correo vía Kit y deje al suscriptor en una lista para números posteriores.

**Arquitectura:** Sitio estático de un solo `index.html`. Se añade una sección `#checklist` entre `#agenda` y `#ebook` que reutiliza el CSS existente de `.ebook`, un formulario que envía el correo al endpoint del formulario de Kit con `fetch()`, y un evento de píxel `CompleteRegistration` distinguido por `content_name`. Kit aloja el PDF y lo entrega tras el doble opt-in; la landing nunca enlaza al archivo.

**Stack:** HTML/CSS/JS a mano, sin build ni dependencias. Kit (plan gratuito) para lista y envío. Píxel de Meta ya instalado.

**Spec:** `docs/superpowers/specs/2026-09-06-newsletter-checklist-kit-design.md`

## Restricciones globales

- **Nunca disparar el evento `Lead`** desde esta sección. `Lead` está reservado a quien declara presupuesto calificado; contaminarlo arruina la optimización de los anuncios.
- El evento es `CompleteRegistration` con `content_name: 'Checklist 12 preguntas'` — exactamente ese texto, porque sobre él se filtra la conversión personalizada en Meta.
- **No puede haber enlace de descarga del PDF en la página.** Vaciaría el doble opt-in y no habría lista.
- **El PDF no se sube a Hostinger.** Lo aloja Kit. En el servidor solo se actualiza `index.html`.
- **El formulario del ebook no se toca.** Sigue con Web3Forms.
- No se añade la sección al menú de cabecera (competiría con "Agenda tu diagnóstico"); solo al pie.
- Las tildes y la ñ van correctas en todo el texto visible.

## Cómo se prueba aquí

No hay framework de pruebas: es un HTML estático y meterle uno sería desproporcionado. La verificación de cada tarea es una comprobación concreta en el navegador, con su resultado esperado escrito.

```bash
cd landing
python3 -m http.server 8777
# abrir http://localhost:8777
```

**Trampa documentada en el README:** abrir la landing en local **dispara eventos reales** del píxel. Antes de probar en local, desactívalo; el propio código trae el interruptor.

```bash
# desactivar el píxel para pruebas locales
sed -i '' "s/window.META_PIXEL_ID = '2291536674928306';/window.META_PIXEL_ID = 'REEMPLAZA_ID_PIXEL';/" index.html
# restaurarlo ANTES de cualquier commit
sed -i '' "s/window.META_PIXEL_ID = 'REEMPLAZA_ID_PIXEL';/window.META_PIXEL_ID = '2291536674928306';/" index.html
git diff --stat index.html   # debe estar vacío respecto al píxel
```

Abrir el archivo con doble clic (`file://`) no sirve: rompe el embed de Calendly.

---

### Tarea 0: Preparar el formulario en Kit

Trabajo manual en el panel de Kit (en inglés). Todo lo demás depende de esto: sin la URL del formulario no se puede probar ni programar nada.

**Archivos:** ninguno. Es configuración en Kit.

**Produce:** la **URL de acción del formulario** (`https://app.kit.com/forms/<ID>/subscriptions`), que consumen la Tarea 1 y la Tarea 3.

- [ ] **Paso 1: Crear el formulario**

En Kit → *Grow* → *Landing Pages & Forms* → *Create new* → **Form** → formato *Inline*. El diseño da igual: la landing no va a usar el aspecto de Kit, solo su endpoint. Nómbralo `Checklist 12 preguntas`.

- [ ] **Paso 2: Subir el PDF y activar la entrega**

**Settings** → pestaña **Confirmation email**:

1. Dejar marcada **Send confirmation email**.
2. Dejar **Auto-confirm new subscribers** SIN marcar: auto-confirmar salta la confirmación y con ella se cae la entrega.
3. En **After confirming redirect to:** elegir **Download** (viene en *URL*) y subir `checklist-12-preguntas.pdf`.

Es el mecanismo que hace todo el trabajo: se configura por formulario y **no** consume la única automatización del plan gratuito.

- [ ] **Paso 3: Escribir el correo en español**

Kit lo trae en inglés. Hay que reescribirlo: el suscriptor es un productor o tostador colombiano.

**Es un solo correo, no dos.** Kit entrega el PDF en la redirección posterior al clic de confirmación, así que este correo carga toda la promesa. Se edita con **Edit Email Contents**. Asunto: `Confirma tu correo y recibe el checklist`

```
Hola:

Pediste el checklist de 12 preguntas para saber si tu marca de café
está lista para salir al mercado.

Confirma tu correo con el botón de abajo y te lo enviamos enseguida.

Marcas al Grano®
```

No hace falta un segundo correo con la invitación a agendar: la última página del propio PDF ya cierra con *"si descubriste que todavía existen preguntas sin resolver, conversemos"*. Si algún día se quiere reforzar, sería un *broadcast* aparte, no parte de esta entrega.

- [ ] **Paso 4: Sacar la URL de acción**

En el formulario → *Publish* → *Embed* → pestaña **HTML**. En el código pegado, copiar el valor del atributo `action`:

```html
<form action="https://app.kit.com/forms/1234567/subscriptions" method="post">
```

Esa URL es la que va a `CONFIG.checklistEndpoint`. **No es un secreto**: viaja en el HTML del navegador, igual que la clave de Web3Forms. Aun así, no pongas ahí ninguna clave de API de Kit — el repo es público.

- [ ] **Paso 5: Probar el circuito desde el propio Kit**

Antes de tocar la landing, suscribirse desde la página alojada del formulario que da Kit, con un correo real. Debe llegar el correo de confirmación, y al hacer clic debe descargarse el PDF y abrir. Si esto falla, el problema es de Kit y no tiene sentido seguir.

---

### Tarea 1: Averiguar si Kit acepta el envío desde el navegador

Es el riesgo técnico del que depende el código de la Tarea 3. Se resuelve probando, no discutiendo. **Bloqueada hasta terminar la Tarea 0**, que produce la URL de acción del formulario.

**Archivos:**
- Ninguno. Es una prueba desechable.

**Produce:** la decisión entre la **variante A** (`fetch` normal, se lee la respuesta) y la **variante B** (`fetch` con `mode:'no-cors'`, éxito optimista). La Tarea 3 usa una u otra.

- [ ] **Paso 1: Levantar el servidor local**

```bash
cd landing && python3 -m http.server 8777
```

- [ ] **Paso 2: Probar el envío desde la consola del navegador**

Abrir `http://localhost:8777`, abrir DevTools → Console y pegar, sustituyendo el ID del formulario y usando **un correo real al que tengas acceso**:

```js
const b = new FormData();
b.append('email_address', 'tu-correo-real@ejemplo.com');
const r = await fetch('https://app.kit.com/forms/XXXXXXX/subscriptions', {method:'POST', body:b});
console.log('status', r.status, await r.text());
```

- [ ] **Paso 3: Anotar el resultado**

| Lo que ves | Significa | Variante |
| --- | --- | --- |
| Un `status` y un cuerpo JSON | CORS permitido | **A** |
| `TypeError: Failed to fetch` + error de CORS en consola | CORS bloqueado | **B** |

- [ ] **Paso 4: Comprobar si el suscriptor llegó igual**

Mirar el panel de Kit. Con la variante B la petición **sale igual** aunque no se pueda leer la respuesta: si el correo aparece en Kit, el camino es válido.

- [ ] **Paso 5: Dejar constancia**

Escribir la variante elegida (A o B) en el mensaje del commit de la Tarea 3. Si sale B, comprobar también que el correo de confirmación de Kit llegó a la bandeja.

> Si ninguna de las dos registra al suscriptor, **para y avisa**. La salida sería incrustar el embed JS propio de Kit, y eso cambia el diseño de la Tarea 2.

---

### Tarea 2: La sección, sin lógica de envío

Solo marcado y estilo. El formulario todavía no envía nada; así se revisa lo visual sin mezclarlo con el comportamiento.

**Archivos:**
- Modificar: `landing/index.html` — bloque CSS `EBOOK` (añadir una regla al final) y una sección nueva justo **antes** del comentario `<!-- ══════════════ EBOOK ══════════════ -->`.
- Modificar: `landing/index.html` — `<nav class="f-links">` del pie.
- Renombrar: `Checklist 12 preguntas antes de lanzar o renovar tu marca de café.pdf` → `checklist-12-preguntas.pdf`.

**Consume:** las clases existentes `.sec`, `.grain`, `.wrap`, `.ebook`, `.ebook-art`, `.ebook-form`, `.eyebrow`, `.lead`, `.note`, `.btn`, `.rv`.

**Produce:** los ids `checklistForm` y `checklistMsg`, que la Tarea 3 usa desde JS.

- [ ] **Paso 1: Renombrar el PDF y versionarlo**

El nombre actual tiene espacios y una tilde: rompe URLs y complica cualquier script.

```bash
cd landing
git mv "Checklist 12 preguntas antes de lanzar o renovar tu marca de café.pdf" checklist-12-preguntas.pdf 2>/dev/null \
  || mv "Checklist 12 preguntas antes de lanzar o renovar tu marca de café.pdf" checklist-12-preguntas.pdf
git add checklist-12-preguntas.pdf
```

- [ ] **Paso 2: Añadir la regla de CSS**

Al final del bloque `EBOOK` del `<style>`, justo después de la línea `@media(max-width:800px){.ebook{grid-template-columns:1fr;gap:2rem}}`:

```css
#checklist .consent{font-size:.72rem;color:var(--dim);margin-top:.55rem;max-width:44ch}
```

- [ ] **Paso 3: Insertar la sección**

Justo **antes** de `<!-- ══════════════ EBOOK ══════════════ -->`:

```html
<!-- ══════════════ CHECKLIST (NEWSLETTER) ══════════════ -->
<section class="sec grain" id="checklist">
  <div class="wrap">
    <div class="ebook rv">
      <div class="ebook-art" aria-hidden="true">
        <p class="t">12 preguntas<em>antes de lanzar</em></p>
        <p class="s">Ponle una nota sobre 12 a tu marca y descubre si es momento de diseñar, de corregir o de detenerte.</p>
        <p class="m">MARCAS AL GRANO®</p>
      </div>
      <div>
        <span class="eyebrow">Autodiagnóstico gratuito</span>
        <h2>¿Tu marca está lista para salir al mercado?</h2>
        <p class="lead">Responde 12 preguntas y ponle una nota sobre 12 a tu marca. En diez minutos vas a saber si es momento de diseñar, de corregir, o de detenerte antes de gastar en empaques que no funcionan.</p>
        <form class="ebook-form" id="checklistForm" novalidate>
          <input type="email" name="email" placeholder="Tu correo electrónico" required aria-label="Correo electrónico">
          <button type="submit" class="btn">Recibir el checklist</button>
        </form>
        <p class="note" id="checklistMsg">16 páginas · PDF · Te llega por correo</p>
        <p class="note consent">Al suscribirte quedas en la lista de Marcas al Grano. Puedes darte de baja cuando quieras.</p>
      </div>
    </div>
  </div>
</section>
```

La sección lleva `sec grain` **sin `alt`**, mientras que el ebook lleva `sec alt grain`. Así las dos secciones seguidas alternan de fondo en vez de quedar planas.

- [ ] **Paso 4: Añadir el enlace del pie**

En `<nav class="f-links">`, entre `Casos` y `FAQ`:

```html
        <a href="#checklist">Checklist</a>
```

- [ ] **Paso 5: Comprobar en el navegador**

Levantar `python3 -m http.server 8777` y verificar, en este orden:

1. La sección aparece **entre** el bloque de agendar y el del ebook.
2. En escritorio: tarjeta a la izquierda, texto y formulario a la derecha.
3. Estrechando la ventana por debajo de 800 px, la tarjeta se pone encima y el formulario debajo, sin desbordes horizontales.
4. El aviso de suscripción se ve más pequeño y apagado que la nota de arriba.
5. El enlace `Checklist` del pie baja a la sección.
6. La sección del ebook, justo debajo, se sigue viendo con su fondo alterno.

- [ ] **Paso 6: Commit**

```bash
cd landing
git add index.html checklist-12-preguntas.pdf
git commit -m "feat: seccion del checklist entre agendar y el ebook

Reutiliza el patron visual de .ebook. El formulario todavia no envia:
la logica va en el commit siguiente. Renombra el PDF a un nombre sin
espacios ni tildes."
```

---

### Tarea 3: Que el formulario suscriba de verdad

**Archivos:**
- Modificar: `landing/index.html` — objeto `CONFIG` (añadir una clave tras `ebookFile`) y bloque `<script>` final (añadir un bloque tras el de `EBOOK`).

**Consume:** `checklistForm` y `checklistMsg` de la Tarea 2; los helpers ya existentes `window.metaTrack(evento, params)` y `window.metaIdentity(correo, telefono)`; `CONFIG.whatsapp`; la variante A o B decidida en la Tarea 1.

- [ ] **Paso 1: Añadir la clave a CONFIG**

Después de `ebookFile: 'ebook-marcas-al-grano.pdf',`:

```js
  // URL de acción del formulario de Kit que entrega el checklist.
  // Sale del embed del formulario en app.kit.com. Vacío = no envía nada.
  checklistEndpoint: 'https://app.kit.com/forms/XXXXXXX/subscriptions',
```

Sustituir `XXXXXXX` por el ID real del formulario. **No es un secreto** — viaja en el HTML, igual que la clave de Web3Forms.

- [ ] **Paso 2: Añadir el bloque de JS**

Justo después del bloque `EBOOK` del `<script>` final, antes del comentario `VARIOS`. Los nombres empiezan por `ck` para no chocar con nada: recuerda que `window.track` ya chocó una vez con un `const track`.

**Si la Tarea 1 dio la variante A (CORS permitido):**

```js
/* ============================================================
   CHECKLIST (NEWSLETTER)
   ============================================================ */
const ckForm = document.getElementById('checklistForm');
const ckMsg  = document.getElementById('checklistMsg');
const ckNota = ckMsg.textContent;
ckForm.addEventListener('submit', async e => {
  e.preventDefault();
  const input = ckForm.email;
  if (!/^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(input.value)) {
    ckMsg.textContent = 'Escribe un correo válido.';
    ckMsg.style.color = '#e08a8a';
    input.focus();
    return;
  }
  const btn = ckForm.querySelector('button');
  btn.disabled = true;
  btn.textContent = 'Enviando…';
  ckMsg.textContent = ckNota;
  ckMsg.style.color = '';

  let ok = true;
  if (CONFIG.checklistEndpoint) {
    try {
      const cuerpo = new FormData();
      cuerpo.append('email_address', input.value);
      const r = await fetch(CONFIG.checklistEndpoint, {method: 'POST', body: cuerpo});
      ok = r.ok;
    } catch (_) { ok = false; }
  }

  if (!ok) {
    btn.disabled = false;
    btn.textContent = 'Recibir el checklist';
    ckMsg.innerHTML = `No pudimos suscribirte. <a href="https://wa.me/${CONFIG.whatsapp}" target="_blank" rel="noopener" style="color:var(--accent-br)">Escríbenos por WhatsApp</a> y te lo enviamos.`;
    ckMsg.style.color = '#e08a8a';
    return;
  }

  metaIdentity(input.value, '');
  metaTrack('CompleteRegistration', {content_name: 'Checklist 12 preguntas', status: true});

  ckForm.style.display = 'none';
  ckMsg.style.color = 'var(--accent-br)';
  ckMsg.textContent = 'Revisa tu correo: te enviamos un enlace para confirmar. Al confirmar se descarga tu checklist.';
});
```

**Si la Tarea 1 dio la variante B (CORS bloqueado):** el mismo bloque, cambiando solo la llamada a `fetch` por esta, porque con `no-cors` la respuesta es opaca y `r.ok` siempre sería `false`:

```js
      const cuerpo = new FormData();
      cuerpo.append('email_address', input.value);
      await fetch(CONFIG.checklistEndpoint, {method: 'POST', mode: 'no-cors', body: cuerpo});
```

Con la variante B solo se detecta el fallo si la red se cae del todo; un ID de formulario equivocado pasaría por éxito. Por eso el Paso 4 comprueba el panel de Kit, no la pantalla.

- [ ] **Paso 3: Probar los caminos que no tocan la red**

Con el píxel desactivado (`REEMPLAZA_ID_PIXEL`) y el servidor local:

| Qué haces | Qué debe pasar |
| --- | --- |
| Enviar vacío | `Escribe un correo válido.` en rojo, el foco vuelve al campo |
| Escribir `hola@` y enviar | Mismo mensaje de error |
| Poner `checklistEndpoint: ''` y enviar un correo válido | Éxito directo: el formulario desaparece y sale el mensaje de "Revisa tu correo" |

Devolver `checklistEndpoint` a su valor real antes de seguir.

- [ ] **Paso 4: Probar el envío real**

Con el endpoint real y **un correo al que tengas acceso**: enviar, y comprobar las tres cosas.

1. El formulario desaparece y sale `Revisa tu correo: te enviamos un enlace para confirmar. Al confirmar se descarga tu checklist.`
2. El suscriptor aparece en el panel de Kit.
3. Llega el correo de confirmación.

- [ ] **Paso 5: Comprobar que el ebook sigue vivo**

Enviar un correo en el formulario del ebook. Debe seguir mostrando su enlace `Abre tu ebook aquí →`. Es el mismo archivo y los dos bloques comparten clases de CSS.

- [ ] **Paso 6: Restaurar el píxel y comprobar que no queda basura**

```bash
cd landing
sed -i '' "s/window.META_PIXEL_ID = 'REEMPLAZA_ID_PIXEL';/window.META_PIXEL_ID = '2291536674928306';/" index.html
grep -n "META_PIXEL_ID = " index.html          # debe mostrar el ID real
grep -n "XXXXXXX" index.html || echo "sin marcadores pendientes"
```

- [ ] **Paso 7: Commit**

```bash
cd landing
git add index.html
git commit -m "feat: el checklist suscribe a Kit y marca el evento

Anade checklistEndpoint a CONFIG y la logica del formulario: validacion,
estados de envio, respaldo por WhatsApp si falla, y CompleteRegistration
con content_name 'Checklist 12 preguntas'. No dispara Lead: ese evento
queda para los leads con presupuesto calificado.

Variante de envio usada: <A o B, segun la Tarea 1>."
```

---

### Tarea 4: Dejar el README al día

El README es el único sitio donde vive el porqué de esta landing. Si no se actualiza, el siguiente que la retome va a repetir trampas ya pisadas.

**Archivos:**
- Modificar: `landing/README.md`

- [ ] **Paso 1: Añadir Kit a la tabla de integraciones**

En la tabla de *Integraciones*, tras la fila de Formularios:

```markdown
| **Kit** (newsletter) | Formulario `<ID>` · plan gratuito | Entrega el checklist por doble opt-in y guarda la lista. El PDF lo aloja Kit, no el hosting |
```

- [ ] **Paso 2: Añadir una subsección sobre los límites de Kit**

Después de *Web3Forms: límites que importan*, en el mismo tono:

```markdown
### Kit: lo que da el plan gratuito

- **10.000 suscriptores y envíos ilimitados**, pero **un solo formulario y una
  sola automatización**. El ebook sigue en Web3Forms justamente para no gastar
  ese formulario: llevarlo también a Kit obliga a pagar.
- **Su marca va al pie de cada correo.** No se quita sin pagar.
- **La entrega es por doble opt-in**: el suscriptor recibe un correo de
  confirmación y solo al hacer clic le llega el PDF. Por eso la landing **no**
  ofrece descarga inmediata: si la ofreciera, nadie confirmaría y no habría lista.
- La entrega se configura **por formulario** (*Settings → Confirmation email*,
  con *After confirming redirect to: Download*), no por lista, y no consume la
  única automatización del plan. Es **un solo correo**: el PDF llega en la
  redirección posterior al clic de confirmación.
- **El panel está en inglés**, aunque los correos al suscriptor van en español.
```

- [ ] **Paso 3: Actualizar el mapa de secciones**

```
hero → video (VSL) → problema → cita → programas → metodología
     → casos → testimonios → agenda → checklist → ebook → FAQ → footer
```

Y añadir a la lista de decisiones no obvias:

```markdown
- **El checklist va después de agendar y antes del ebook.** Después, por lo
  mismo que el ebook: encima del formulario le da salida fácil a quien ya venía
  convencido. Antes del ebook, porque es la pieza más fuerte — hace que el
  lector se ponga una nota sobre 12 y cierra invitando a conversar, así que es
  la única que devuelve gente al calendario.
- **El PDF del checklist no está en el servidor.** Lo aloja Kit, para que el
  enlace no sea adivinable. El archivo del repo es solo la copia versionada, y
  como el repo es público el bloqueo es blando: quien lo busque en GitHub lo
  encuentra. Se acepta, igual que con el ebook.
```

- [ ] **Paso 4: Añadir el evento a la tabla del píxel**

```markdown
| `CompleteRegistration` | Suscripción al checklist (`content_name: Checklist 12 preguntas`) | Estándar |
```

Y una línea bajo la tabla:

```markdown
**El checklist no dispara `Lead`** a propósito: `Lead` está reservado a quien
declara presupuesto calificado. Meter ahí suscriptores de un PDF gratuito haría
que Meta optimice hacia gente sin dinero.
```

- [ ] **Paso 5: Actualizar los pendientes**

Marcar como cubierto el punto 1 (probar en teléfono real) **solo si la Tarea 5 se hizo desde un móvil**, y añadir:

```markdown
- **Dominio remitente propio en Kit.** Hoy los correos salen desde la
  infraestructura de Kit. Configurar `somosmarcasalgrano.com` con sus registros
  DNS en Hostinger mejora que no caigan en spam.
```

- [ ] **Paso 6: Commit**

```bash
cd landing
git add README.md
git commit -m "docs: README al dia con Kit, el checklist y su evento"
```

---

### Tarea 5: Publicar y verificar en producción

Ni el correo de Kit ni el píxel se comprueban en local: Kit solo dispara en sitios en producción y el píxel en local ensucia datos reales.

**Archivos:** ninguno. Es despliegue y verificación.

- [ ] **Paso 1: Subir `index.html`**

Por el Administrador de archivos de Hostinger, a `public_html/`. **Solo `index.html`.** El PDF no va al servidor: lo entrega Kit.

- [ ] **Paso 2: Comprobar que se ve lo nuevo**

Abrir `https://somosmarcasalgrano.com` y confirmar que la sección aparece entre agendar y el ebook. El `.htaccess` no cachea el `index.html`, así que el cambio debe verse de inmediato.

- [ ] **Paso 3: Recorrer el circuito completo desde un teléfono real**

Es el pendiente número 1 del README y aquí se cubre. Con un correo real:

1. Suscribirse desde el móvil.
2. Llega el correo de confirmación.
3. Al confirmar, el clic lleva a la descarga del PDF.
4. El PDF abre y son 16 páginas.
5. El suscriptor aparece en el panel de Kit.

- [ ] **Paso 4: Comprobar el evento en Meta**

En el Administrador de eventos, tras suscribirse: aparece `CompleteRegistration` con `content_name: Checklist 12 preguntas`. Confirmar también que **no** se disparó ningún `Lead`.

- [ ] **Paso 5: Crear la conversión personalizada**

En el Administrador de anuncios, una conversión personalizada sobre `CompleteRegistration` filtrada por ese `content_name`. Así el checklist y el ebook quedan separados en los informes.

- [ ] **Paso 6: Comprobar que no se rompió nada**

En producción: el formulario del ebook sigue entregando su enlace, el modal de agendar sigue abriendo con el programa correcto desde cada botón "Es para mí", y el embed de Calendly sigue cargando.

- [ ] **Paso 7: Anotar el estado**

Si algo del circuito falló, escribirlo en los *Pendientes* del README antes de dar la tarea por cerrada.
