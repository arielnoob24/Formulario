# Auditoría del formulario

**Fecha:** 2026-09-08  
**Agente:** Auditor OpenCode  
**Alcance:** `index.html`, `styles.css`, configuración de despliegue y estado del repositorio.  
**Criterios:** funcionalidad, accesibilidad POUR/WCAG AAA, seguridad, validación, UX responsive y mantenimiento.

## Hallazgos

### Crítico

Ninguno identificado.

### Alto

1. **El formulario no tiene destino funcional**
   - Ubicación: [index.html](index.html#L23)
   - `action="#"` envía el `POST` a la propia página. En GitHub Pages no existe un backend que procese los datos, archivos adjuntos o consentimiento.
   - **Impacto:** el usuario puede creer que la consulta fue enviada, pero los datos no llegan a ningún destinatario. No hay confirmación, error ni recuperación.
   - **Recomendación:** integrar un endpoint HTTPS real, servicio de formularios o función serverless. Añadir estados de envío exitoso/error y validación servidor.

2. **El despliegue automático de GitHub Pages fue eliminado**
   - Ubicación: historial Git, commit `d6b6776`; se eliminó `.github/workflows/pages.yml`, que existía en `a7772ad`.
   - **Impacto:** los nuevos cambios ya no se publican automáticamente mediante GitHub Actions. Solo funcionaría si Pages está configurado por otra vía, como despliegue desde una rama.
   - **Recomendación:** confirmar la fuente configurada en GitHub Pages. Si se esperaba Actions, restaurar un workflow equivalente y verificar permisos, entorno y artefacto publicado.

3. **La interfaz muestra funciones que no funcionan**
   - Ubicación: [index.html](index.html#L127)
   - `Guardar borrador` es un botón `type="button"` sin JavaScript ni destino.
   - **Impacto:** acción aparentemente disponible pero inoperante, especialmente problemática para usuarios que rellenan formularios largos.
   - **Recomendación:** implementar almacenamiento local claramente indicado o eliminar el botón hasta disponer de una función real.

### Medio

4. **Existen dos controles de envío**
   - Ubicación: [index.html](index.html#L129) y [index.html](index.html#L130)
   - Hay un `input type="image"` y un `button type="submit"`.
   - **Impacto:** duplica la acción de envío, añade una parada innecesaria a la navegación por teclado y puede confundir a usuarios de lector de pantalla. El botón-imagen también puede mostrar un recurso roto.
   - **Recomendación:** conservar un único botón submit accesible, preferiblemente el botón textual.

5. **El SVG embebido usa un namespace incorrecto**
   - Ubicación: [index.html](index.html#L129)
   - El namespace efectivo queda como `%68%74%74%70...` en lugar de `http://www.w3.org/2000/svg`.
   - **Impacto:** el XML es sintácticamente válido, pero el navegador puede no reconocerlo como SVG y mostrar una imagen rota o solo su texto alternativo.
   - **Recomendación:** corregir la codificación del `data:` URI o eliminar el control de imagen duplicado.

6. **El ejemplo de URL no coincide con la validación nativa**
   - Ubicación: [index.html](index.html#L64)
   - `type="url"` muestra `ejemplo.com`, pero normalmente el navegador exige un esquema como `https://ejemplo.com`.
   - **Impacto:** el usuario puede copiar exactamente el ejemplo y recibir un error de validación.
   - **Recomendación:** usar `https://ejemplo.com` como placeholder o normalizar la URL antes de validarla.

7. **El campo de código de seguimiento está modelado como contraseña**
   - Ubicación: [index.html](index.html#L59)
   - Usa `name="password"`, `type="password"` y `autocomplete="new-password"` aunque la etiqueta dice “Código de seguimiento”.
   - **Impacto:** puede activar gestores de contraseñas, ocultar innecesariamente el valor y asociar datos de consulta con credenciales. Si el valor se transmite cuando exista backend, el nombre también resulta engañoso.
   - **Recomendación:** utilizar un nombre específico, por ejemplo `tracking-code`, y `type="text"` salvo que realmente sea un secreto.

8. **Restricciones temporales frágiles y campos de fecha incoherentes**
   - Ubicación: [index.html](index.html#L77), [index.html](index.html#L81) y [index.html](index.html#L85)
   - La fecha mínima está fijada a `2026-01-01`, mientras que `datetime-local` no tiene restricciones equivalentes.
   - **Impacto:** se vuelve obsoleta con el tiempo y permite combinaciones contradictorias entre fecha, hora y fecha-hora exacta.
   - **Recomendación:** generar límites dinámicamente o eliminarlos si no son requisitos; definir claramente qué campo se utiliza y validar relaciones en servidor.

9. **Los límites de archivo solo son indicativos**
   - Ubicación: [index.html](index.html#L101)
   - `accept` restringe la selección del navegador, pero no valida seguridad, contenido real, tamaño ni extensión en un servidor.
   - **Impacto:** si se conecta posteriormente a un backend, se podrían subir archivos no permitidos o excesivamente grandes.
   - **Recomendación:** validar MIME real, extensión, tamaño, contenido y almacenamiento aislado en el servidor. No confiar en `accept`.

10. **Contraste insuficiente para el nivel WCAG AAA solicitado**
    - Ubicación: [styles.css](styles.css#L69), [styles.css](styles.css#L87) y [styles.css](styles.css#L97)
    - `#b7e3d5` sobre `#0d6259`: aproximadamente `5.14:1`.
    - `#d8f1e9` sobre `#0d6259`: aproximadamente `6.08:1`.
    - **Impacto:** pasan los mínimos habituales de WCAG AA para texto normal, pero no alcanzan AAA, que exige `7:1`.
    - **Recomendación:** oscurecer los textos secundarios o aclarar/ajustar el fondo verde.

11. **El indicador de foco no alcanza contraste suficiente**
    - Ubicación: [styles.css](styles.css#L276) y [styles.css](styles.css#L410)
    - `#f2a65a` sobre fondo blanco tiene aproximadamente `2.02:1`.
    - **Impacto:** el foco puede ser difícil de percibir y no alcanza el criterio de contraste de indicador no textual de WCAG 2.2.
    - **Recomendación:** usar un color de foco con al menos `3:1` respecto al fondo y mantener grosor suficiente.

12. **Los bordes de los campos son demasiado débiles como indicador visual**
    - Ubicación: [styles.css](styles.css#L170)
    - `#dce3e7` sobre fondos casi blancos tiene contraste muy bajo.
    - **Impacto:** usuarios con baja visión pueden no identificar claramente los límites de los controles antes de enfocarlos.
    - **Recomendación:** aumentar el contraste del borde o usar una diferencia visual adicional que alcance al menos `3:1`.

13. **El estado inválido depende principalmente del color**
    - Ubicación: [styles.css](styles.css#L200)
    - El campo inválido solo cambia el color del borde.
    - **Impacto:** la información puede no percibirse para personas con daltonismo o con modos de alto contraste. No hay mensaje contextual propio ni icono.
    - **Recomendación:** mostrar mensajes asociados mediante `aria-describedby`, mantener `aria-invalid` actualizado y añadir una señal no basada exclusivamente en color.

### Bajo

14. **No hay adaptación específica para movimiento reducido**
    - Ubicación: [styles.css](styles.css#L174), [styles.css](styles.css#L256) y [styles.css](styles.css#L263)
    - Hay transiciones y desplazamiento vertical en hover, sin `prefers-reduced-motion`.
    - **Impacto:** riesgo menor para usuarios sensibles al movimiento.
    - **Recomendación:** desactivar o reducir transiciones bajo `@media (prefers-reduced-motion: reduce)`.

15. **No se proporciona una política de privacidad enlazada**
    - Ubicación: [index.html](index.html#L120)
    - El consentimiento menciona tratamiento de datos, pero no enlaza a información de privacidad, conservación o derechos.
    - **Impacto:** insuficiente transparencia para un formulario que recopila datos personales y potencialmente archivos.
    - **Recomendación:** enlazar la política aplicable y explicar finalidad, base legal, conservación y contacto responsable.

## Aspectos sin problemas relevantes detectados

- Los controles visibles tienen `label` asociado; el grupo de prioridad está agrupado semánticamente.
- No se encontraron IDs duplicados ni referencias `for` rotas.
- `lang="es"`, `title`, `meta viewport` y `meta description` están presentes.
- El orden del DOM es razonable para teclado y lectores de pantalla.
- Se usan tipos adecuados en varios campos: `email`, `tel`, `url`, `date`, `time`, `file`, `number`, `range`, `radio` y `checkbox`.
- El consentimiento es obligatorio y está correctamente asociado a su etiqueta.
- No hay scripts, dependencias externas, credenciales ni contenido activo de terceros en los archivos auditados.
- El diseño tiene breakpoints para tabletas y móviles, y no se observó una causa estática evidente de desbordamiento horizontal en los anchos definidos.

## Preguntas o supuestos

- ¿El formulario debe enviar realmente los datos o es únicamente una maqueta visual?
- ¿GitHub Pages está configurado para desplegar desde la rama `main`, o se esperaba el workflow eliminado?
- ¿“Código de seguimiento” es un secreto real? Si no lo es, no debería tratarse como contraseña.
- ¿La fecha mínima `2026-01-01` representa una regla de negocio o fue un valor de demostración?
- No se evaluaron políticas del servidor porque no existe backend en el repositorio.

## Validaciones ejecutadas

- Revisión de todos los archivos versionados y del historial de despliegue.
- Comprobación del árbol de trabajo: `index.html` y `styles.css` permanecen sin cambios.
- `git diff --check`: sin errores.
- Comprobación estructural de IDs, labels, referencias locales y controles.
- Decodificación e inspección del SVG embebido.
- Cálculo de ratios de contraste para textos, foco y controles.
- No hay validador HTML/CSS ni navegador automatizado instalado localmente; por eso no se ejecutó una validación W3C ni una prueba visual real con múltiples viewport.
