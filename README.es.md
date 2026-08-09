# Tutorial completo para crear un método de entrada de lengua construida (edición en español)

🌐 [中文](README.md) | [English](README.en.md) | [日本語](README.ja.md) | **Español** | [Русский](README.ru.md)

Este tutorial explica de principio a fin cómo crear un método de entrada para tu propia lengua construida (conlang), en tres grandes bloques: escribir código SVG, crear fuentes TTF/OTF y configurar el mapeo de teclas y el teclado en pantalla en Keyman Developer.

Hasta donde sé, no existe una guía completa que detalle estos tres pasos juntos. Además, las herramientas actuales para editar SVG y fabricar TTF/OTF suelen no ser lo bastante amigables para principiantes. Por eso, con *vibe coding* (desarrollo asistido por IA), creé un editor de código SVG y una aplicación para fabricar fuentes TTF/OTF. Toda la demostración del tutorial se hace con estos dos programas; antes de leer y practicar, descárgalos desde los enlaces siguientes.

1. **Editor de código SVG**  
   [Descargar SVG-Code-Editor-Setup.exe](https://github.com/ajt071220-netizen/conlang-ime-tutorial-zh/releases/download/v1.0.0/SVG-Code-Editor-Setup.exe)

2. **Aplicación para fuentes TTF/OTF**  
   [Descargar SVGFont-Studio-Setup.exe](https://github.com/ajt071220-netizen/conlang-ime-tutorial-zh/releases/download/v1.0.0/SVGFont-Studio-Setup.exe)

## I. Crear glifos con SVG

Antes de empezar necesitamos SVG, un formato de gráficos vectoriales. Dibujaremos nuestros caracteres con el propio código SVG. SVG (*Scalable Vector Graphics*) es el formato estándar de los métodos de entrada más usados en el mundo. Las formas descritas en código se dibujan como vectores según la densidad de pantalla, así que siguen nítidas a cualquier tamaño —a diferencia del PNG, que se ve borroso al ampliarse—. El renderizado es controlable y no depende de un archivo de fuente previo; por eso SVG se ha convertido en el estándar de la industria. En este tutorial usamos sobre todo formas y rutas SVG para dibujar nuestros glifos. Las rutas pueden dibujar casi cualquier cosa; cada forma también tiene su propia sintaxis. Algunas formas son engorrosas solo con rutas, así que primero veremos la preparación y varias formas más fáciles con elementos dedicados: esta es la introducción a la sintaxis SVG.

### (1) Tamaño del lienzo y coordenadas

![Tamaño del lienzo y coordenadas](images/02.png)

Configurar el lienzo es la preparación más básica antes de dibujar glifos. Como se ve arriba, consta de tres partes: el espacio de nombres XML (`xmlns` y la larga cadena que sigue), `width` y `height`, y `viewBox`. Importan sobre todo las dos últimas; la primera no hace falta estudiarla a fondo. `width` y `height` son el ancho y el alto del lienzo. Para métodos de entrada, el tamaño recomendado es **1024×1024**, como en la captura: es el estándar de facto del sector. `viewBox` define el sistema de coordenadas. Sus cuatro números significan: los dos ceros iniciales son el origen `(0, 0)`; los dos `1024` siguientes son el ancho y el alto en los ejes x e y. Al dibujar, mantén una proporción 1:1 entre width/height y los rangos x/y del viewBox: si el ancho es 1024, el rango x también debe ser 1024, e igual con la altura. En este programa, si no sabes dónde está un punto, mueve el ratón sobre el lienzo de la derecha; se mostrarán las coordenadas actuales.

### (2) Rectángulo (`rect`)

![Rectángulo 1](images/04.png) ![Rectángulo 2](images/14.png)

En el Rectángulo 1, `rect` es un rectángulo; `x` e `y` son la esquina superior izquierda; `width` y `height` son el ancho y el alto. En el Rectángulo 2, `fill` es el color de relleno. `fill` se usa sobre todo en formas cerradas, pero a veces también hace falta con trazos (lo veremos más adelante). `rx` y `ry` controlan el redondeo horizontal y vertical de las esquinas: imagina cada esquina redonda como parte de una elipse; `rx` y `ry` son sus ejes. Al principio puede costar: dibuja una elipse centrada en la esquina que «corta» el ángulo con una curva tangente. Como las curvas de la sección de rutas, se aprende con práctica.

### (3) Círculo (`circle`)

![Círculo](images/06.png)

En el código de círculo: `circle` es el círculo; `cx` y `cy` son el centro; `r` es el radio.

### (4) Elipse (`ellipse`)

![Elipse](images/08.png)

El código de elipse es parecido al del círculo. `ellipse` es la elipse; la diferencia es que `rx` y `ry` son los radios horizontal y vertical (el mayor actúa como eje mayor y el menor como eje menor), según tu diseño.

### (5) Línea (`line`)

![Línea 1](images/10.png) ![Línea 2](images/12.png) ![Rectángulo 3](images/16.png)

En la Línea 1 hay dos puntos `(x1, y1)` y `(x2, y2)`: inicio y fin. `stroke` es el color del trazo; `stroke-width` el grosor; `stroke-linecap` la forma de los extremos. En la primera imagen se usa `butt` (valor por defecto de SVG: extremos planos). En la segunda, `round` da extremos redondeados. Son las dos opciones más útiles. En el Rectángulo 3 también puedes usar `stroke` y `stroke-width` para resaltar el contorno.

### (6) Uniones de esquina (`stroke-linejoin`)

![Esquina 1](images/18.png) ![Esquina 2](images/20.png) ![Esquina 3](images/22.png)

Las tres imágenes muestran esquinas redonda, aguda y biselada: `round`, `miter` y `bevel`. Importan sobre todo en polilíneas. Con polilíneas, cuida también `stroke` y `stroke-width`, y pon `fill="none"`; si no, el sistema puede intentar rellenar del inicio al final como si fuera una forma cerrada.

### (7) Ruta (`path`)

![Ruta 1](images/24.png) ![Ruta 2](images/26.png) ![Ruta 3](images/28.png)

`path` es el elemento SVG más importante: con él puedes dibujar casi cualquier diseño. En la primera imagen, `M` significa *move to*: mueve el lápiz a ese punto como inicio, por ejemplo `(100, 100)`. `L` significa *line to*: traza una recta hasta ese punto. La segunda imagen muestra curvas —lo más importante de las rutas—: una cúbica de Bézier. `M` sigue siendo el inicio; `C` introduce dos puntos de control. Esos puntos no están sobre la curva; piénsalos como imanes que doblan un alambre. Cuanto más cerca del pliegue, más aguda la curva; cuanto más lejos, más suave. Una cúbica tiene dos puntos de control —p. ej. `(430, 430)` y `(974, 430)`— y un punto final; el inicio sigue siendo `M`. La tercera imagen es una cuadrática de Bézier con `Q`, un solo punto de control y un final. Los puntos de control requieren práctica; conviene experimentar.

Con esto terminan los fundamentos de SVG. Antes de la etapa 2: sigue con rigor el formato de las capturas —mayúsculas/minúsculas, `<` y `/>` en cada etiqueta, y comillas dobles rectas `""`—. Comprueba sobre todo que la declaración de la primera línea empiece y termine bien con `<` y `>`; si falta un solo carácter, a menudo se rompe la vista previa. Los elementos anteriores se pueden combinar: por ejemplo, una recta con `path` y un círculo con `circle` al lado. Escribe el código de las formas entre la declaración SVG y el cierre `</svg>`. Mantén completos ambos; un solo símbolo que falte puede detener la vista previa y dar error.

## II. Crear archivos de fuente TTF/OTF

![Creación de fuente 1](images/30.png)

Abre la aplicación de fuentes como arriba. Primero nombra el documento / archivo de fuente a tu gusto —ese nombre importará después—. Tras ajustar lo inicial, ve a **Gestión de fuentes**, pulsa **Añadir un SVG** y elige el SVG que guardaste en el editor. Tras subirlo, se asigna un punto de código en el Área de Uso Privado (PUA) de Unicode. Unicode es un estándar universal que da un número único a cada carácter de cada lengua y símbolo, para evitar mojibake e intercambiar datos con compatibilidad. Es una de las bases técnicas de los métodos de entrada. La PUA es un bloque reservado sin significado oficial; lo asignan usuarios, instituciones o fabricantes de fuentes.

Después de subir el glifo, ajusta los márgenes laterales (**LSB** y **RSB**) y el avance (advance width). Como referencia aproximada en **1024×1024 UPM** con letras latinas:

- Minúsculas estrechas como `i` y `j`: unos **220–320**
- Un poco más anchas como `f`, `t`, `r`: unos **280–400**
- Minúsculas normales como `a`, `n`, `o`, `e`: unos **480–620**
- Minúsculas anchas como `w`, `m`: unos **750–920**
- Mayúsculas normales: unos **580–720**
- Mayúsculas muy anchas como `W`, `M`: unos **800–1000**

Como regla práctica, cada margen lateral suele ser cerca del **8%–18%** del avance. Como la PUA se asigna en orden, sube tus glifos en el orden de tu alfabeto si puedes. Luego ve a **Generar / Exportar** y pulsa **Generar fuente** para crear tu TTF.

## III. Descargar y usar Keyman Developer y Keyman for Windows

Esta es la parte central —y la más crítica—. En este motor de desarrollo de métodos de entrada combinamos la fuente y terminamos el teclado. Primero descarga ambos programas del sitio oficial:

- [Descargar Keyman Developer](https://keyman.com/developer/download)
- [Descargar Keyman for Windows](https://downloads.keyman.com/windows/stable/18.0.249/keyman-18.0.249.exe)

**Nota:** Si falla el enlace directo, abre la [página oficial de descarga de Keyman for Windows](https://keyman.com/windows/download) y descarga a mano.

Tras instalar, abre Keyman Developer por primera vez, pulsa **New Project**, elige **Keyman keyboard**, y escribe nombre y descripción. Recomiendo la transliteración latina del nombre de tu conlang para ambos. Pulsa **OK**. En el árbol, selecciona **Keyman keyboard** y abre el archivo `.kmn` con el nombre de tu lengua.

Luego, en el menú superior izquierdo **View**, elige **Character Font**. Busca el nombre de familia de tu fuente (**sin** `.ttf`), pulsa **Apply** y luego **OK**. Puede aparecer un error del tipo «el tamaño debe estar entre 0 y 0». Si pasa, ignóralo: tras **Apply**, si **OK** abre el diálogo, cierra el diálogo y el selector de fuente. Más abajo corregiremos la fuente del teclado en pantalla.

Abre la pestaña **Layout**, pasa de **Design** a **Code**, y bajo `group(main) using keys` (hacia la línea 14) escribe reglas desde la línea 15. Cada regla tiene esta forma:

```text
+ [K_A] > U+E000
```

(con espacio después de `+`, después de `]` y alrededor de `>`). `[K_A]` es la tecla física A (la `a` minúscula sin Shift). Asigna Unicode a teclas como quieras: usaremos sobre todo el teclado en pantalla, no el físico. `+` y `>` son obligatorios, con los espacios indicados. Mantén cada tecla emparejada con su Unicode. Puedes usar `[K_B]`, `[K_C]`, etc.; no olvides el guion bajo.

Cuando tengas tantas reglas como letras, abre **On-Screen** bajo Layout y pulsa **Fill from layout**. Eso copia el mapeo, pero **no** hace aparecer solos tus glifos en las teclas. Abre la vista **Code** de On-Screen y busca hacia la línea 8:

```xml
<encoding name="unicode" fontname="Arial" fontsize="-12">
```

Cambia `fontname` de `Arial` al nombre de tu fuente (mantén las comillas) y `fontsize` a **-16** —mantén comillas y el signo menos; solo cambia el número—. Entonces las teclas pueden mostrar tus caracteres. En **Design** puede que sigan en blanco; en **Text** puede verse un espacio vacío. Es normal: son caracteres PUA y a menudo no se renderizan en el diseñador.

Cuando termines, abre **Build** y, en **File actions**, pulsa **Compile Keyboard**. Si **Messages** muestra una línea verde que termina en **built successfully**, compiló bien. Luego ve a **Packaging**, abre el `.kps`, selecciona el teclado y en **Languages** pulsa **Add**. En el primer campo **Tag** escribe una etiqueta de lengua de como máximo tres letras, guarda, y haz **File → Save**. Compila de nuevo: en **File actions**, **Compile Package**. Deberías ver otra vez **Built successfully**.

Abre **Keyman for Windows** y pulsa **Start Keyman**. La ventana puede cerrarse; comprueba que Keyman esté en la bandeja del sistema (flecha junto al reloj). Si está, está en marcha. Vuelve a Keyman Developer, abre **Build** del `.kps` y pulsa **Install**. Al terminar, cambia al nuevo método de entrada, abre el menú de Keyman en la bandeja y elige **On Screen Keyboard**. Debería aparecer un teclado virtual con tus glifos: ya puedes escribir.
