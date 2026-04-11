# Presentación — TI Joaquín González

**Optimización de intervenciones automatizadas contra discursos de odio en redes sociales**  
CEIA FIUBA · Abril 2026

---

## Estructura

```
presentacion/
├── index.html                  ← presentación completa (Reveal.js)
├── images/                     ← imágenes de la tesis
│   ├── eda/esp/                ← gráficos EDA en español
│   └── toxicity/               ← gráficos de evaluación de clasificadores
├── vendor/
│   ├── reveal/                 ← Reveal.js local (reveal.js + reveal.css)
│   └── fonts/                  ← JetBrains Mono local (TTF)
├── Uba_fiuba_ingenieria_logo.png
└── TI_JOAQUIN_GONZALEZ_V10.pdf ← memoria completa
```

---

## Ver la presentación

Abrir `index.html` en cualquier navegador moderno (doble click o arrastrar).

### Navegación

| Acción | Tecla |
|--------|-------|
| Siguiente slide | `→` o `Espacio` |
| Slide anterior | `←` |
| Primer slide | `Home` |
| Último slide | `End` |
| Vista grilla (todos los slides) | `Esc` |
| Pantalla completa | `F` |

---

## Exportar a PDF (simil PPT)

**Requiere Google Chrome.**

### Pasos

1. Abrir Chrome y navegar a la presentación con el parámetro `?print-pdf`:

   ```
   file:///ruta/absoluta/a/presentacion/index.html?print-pdf
   ```

   > El parámetro `?print-pdf` es indispensable. Sin él, Chrome imprime la página como una web continua y los slides se cortan. Con él, Reveal.js convierte cada slide en un bloque de página independiente.

2. Esperar ~3 segundos a que carguen todas las imágenes y fuentes.

3. Presionar `Ctrl+P` (Imprimir).

4. Configurar el diálogo de impresión:

   | Opción | Valor |
   |--------|-------|
   | Destino | **Guardar como PDF** |
   | Diseño | Horizontal |
   | Tamaño de papel | **Personalizado: 33,86 cm × 19,05 cm** |
   | Márgenes | **Ninguno** |
   | Escala | Predeterminada |
   | Gráficos de fondo | **Activado** ← importante para el fondo negro |

5. Guardar como `TI_JOAQUIN_GONZALEZ_presentacion.pdf`.

---

## Video (slide 4)

El slide 4 tiene un placeholder. Cuando el video esté grabado:

1. Colocar el archivo en `video/normsy_demo.mp4`
2. En `index.html`, buscar el comentario `<!-- Reemplazar src con la ruta al video -->` en el slide 4
3. Descomentar el bloque `<video>` y comentar el bloque `.vid-placeholder`

---

## Dependencias locales

Todo está incluido localmente — la presentación funciona sin conexión a internet:

- **Reveal.js 5.1.0** → `vendor/reveal/`
- **JetBrains Mono** (pesos 300–700) → `vendor/fonts/`
