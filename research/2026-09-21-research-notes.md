# Qwen-Image-2.1 — Notas de investigación para una skill de guía de prompts (2026-09-21)

Resumen de investigación primaria para la creación de una Agent Skill de guía de prompts. Fecha de investigación: 2026-09-21 (el día después de la publicación del modelo).

---

## 1. Resumen del modelo

- **Nombre oficial**: Qwen-Image-2.1 (variantes de escritura: "Qwen Image 2.1" / «千问图像2.1»)
- **Desarrollador**: equipo Alibaba Qwen (laboratorio Tongyi de Hangzhou)
- **Fecha de anuncio**: **2026-09-20** (según GitHub News; la documentación de licencia en HF es del mismo día)
- **Posicionamiento**: «el modelo de generación de imágenes open source más potente de Qwen hasta la fecha». Unifica generación y edición en un solo modelo ("小模强效，创改一体")
- **Linaje**: Qwen-Image (2025-08, 20B, Apache 2.0) → Edit-2509 (2025-09) → Edit-2511 / Qwen-Image-2512 / Layered (2025-12) → Qwen-Image-2.0 (2026-02, solo API) → Qwen-Image-3.0/3.0-Pro (2026-07, solo API, cerrado) → **Qwen-Image-2.1 (2026-09-20, 7B, open weights)**

### Advertencias importantes
- La numeración no es intuitiva: 3.0 es más reciente pero es cerrado. El más reciente con pesos abiertos es 2.1
- **La licencia NO es Apache 2.0, sino el "Qwen Research License Agreement"** (gratuito solo para fines no comerciales de investigación y evaluación; el uso comercial requiere negociación individual en model-business@notice.qwencloud.com; la redistribución exige adjuntar la licencia y la atribución)
- En la API Bailian de Alibaba Cloud, a fecha de 2026-09-21 todavía no aparece "qwen-image-2.1 (por ser justo tras el lanzamiento)

## 2. Variantes y formatos de publicación

| Modelo | Contenido | Parámetros |
|---|---|---|
| Qwen/Qwen-Image-2.1 | El modelo principal. Unifica generación + edición | 7B |
| Qwen/Qwen-Image-2.1-PE-T2I | Reescritor de prompts oficial (para generación). Instrucciones cortas → prompt detallado en inglés + `wh_ratio` | 9.4B (basado en Qwen3.5-VL 9B) |
| Qwen/Qwen-Image-2.1-PE-I2I | Reescritor de prompts oficial (para edición). Instrucciones ambiguas → directivas de edición estrictas | Ídem |
| Comfy-Org/Qwen-Image-2.1 | Paquete de pesos para ComfyUI (soporte Day-0) | — |

- Frameworks con soporte Day-0: **Diffusers (recomendado)**, ComfyUI, vLLM-Omni, SGLang, LightX2V
- Por ahora no hay versión Turbo/destilada
- Demo: HF Spaces (Qwen/Qwen-Image-2.1)

## 3. Especificaciones y capacidades

### Arquitectura
- Single-Stream DiT de 32 capas (MMDiT optimizado), 7B
- Codificador de texto: Qwen3-VL 8B / VAE: RGBA de 64 canales, compresión 16×
- Flow Matching (Euler + dynamic shifting), reutilización de Prefix KV Cache, atención de granularidad mixta

### Funciones nuevas respecto a la generación anterior
1. **Unificación de generación + edición** (Qwen-Image y Qwen-Image-Edit en uno)
2. **Imágenes RGBA transparentes nativas** (generación con transparencia, edición de capas transparentes, extracción de sujeto)
3. **Hasta 10 imágenes de referencia como entrada** (composición de varias personas; referencias de personaje, producto, fondo y estilo. El nodo de ComfyUI admite hasta 16)
4. **Edición local**: círculos, anotaciones con pintura o máscaras para delimitar la zona de edición (el equivalente al ControlNet nativo de los antiguos Edit se sustituye por esta forma)
5. **Preservación de identidad de personas y productos**
6. **2K nativo y renderizado de texto de alta definición** (pósters, tipografía, infografías, panorámicas, storyboards)

### Relaciones de aspecto (ancho×alto, 2K)
| Ratio | Resolución |
|---|---|
| 1:1 | 2048×2048 |
| 4:3 / 3:4 | 2400×1792 / 1792×2400 |
| 3:2 / 2:3 | 2528×1696 / 1696×2528 |
| 16:9 / 9:16 | 2752×1536 / 1536×2752 |

- Valores por defecto: 2048×2048 y 40 pasos de inferencia
- El reescritor PE selecciona semánticamente, además de esos 7 ratios, otros como 2:1, 21:9, 9:21, 4:5, 3:1, 5:4, 1:3, 18:39, 9:20, 7:3, 9:5, 5:7; el pipeline mapea al ratio más cercano
- **Diffusers admite `negative_prompt` y `true_cfg_scale` (por defecto 4.0)** (se activan con true_cfg>1 más un negativo). En las recetas destiladas de SGLang/vLLM se usa guidance 1 sin negativo

---

## 4. Puntos clave de la guía oficial de prompts

No existe un único documento de "guía de prompts". Las directrices oficiales están dispersas en:
1. La sección "Prompt Rewriting" del README de GitHub
2. **El `system_prompt.txt` incluido en los repositorios PE-T2I / PE-I2I (de facto, el verdadero cuerpo de la guía oficial de prompts)**
3. La plantilla de imágenes transparentes de la model card

### (A) Generación (T2I) — a partir del system prompt de PE-T2I

**Principio básico**: describir la imagen terminada desde el punto de vista de un «observador», en **un único párrafo largo en inglés (unas 20 frases, 400–500 palabras)**. Tanto las instrucciones cortas como las largas se expanden al mismo tamaño (las cortas se completan inventando la mayor parte).

Estructura en 8 pasos:
1. **Separar elementos fijos y libres**: las cadenas de texto, nombres de objetos, cantidades, colores, posiciones y ratios especificados por el usuario se mantienen **literalmente, palabra por palabra**. Las «notas de uso» (p. ej. "en 4K sin ruido") se reflejan en la descripción pero no se repiten textualmente
2. **Decidir el encuadre**: el ratio va solo en el campo `wh_ratio`; **no escribir ratios, resoluciones ni píxeles en el texto descriptivo**. Por defecto 3:2 horizontal y 2:3 vertical. 1:1 (badges, iconos), 16:9 (cine, presentaciones), 9:16 (móvil, banner vertical), etc. se eligen por semántica
3. **Frase inicial (unas 20 palabras)**: «The image is a ⟨vertical/horizontal/cuadrada⟩ ⟨estilo⟩ ⟨foto/póster/ilustración…⟩ de ⟨sujeto⟩, ⟨fondo y paleta⟩». El sustantivo del medio es obligatorio; el término de estilo se nombra aquí una única vez
4. **Inventario**: asignar a todos los elementos una posición en el encuadre (upper-left, across the top, lower-third, in the centre…). **8–14 frases posicionales (orientativo: 10)**, cubriendo esquinas, bordes y centro por igual
5. **Recorrido del encuadre**: si es una imagen de maquetación, «fondo → franja superior → cuerpo (izquierda → centro → derecha) → franja inferior». Si es un sujeto único, «fondo → colocación → cabeza y rostro → cuerpo y vestuario → objetos en las manos → bordes». **Aproximadamente 1 de cada 3 frases empieza con una frase posicional**
6. **Configuración del texto**: todo lo que se pueda leer se escribe en orden de lectura con la forma `a bold black headline across the top reads "…"`, con el texto original literal entre comillas de cita (el chino se mantiene en chino). Especificar también grosor, color y tamaño. Lo que no debe leerse: "blurred, indistinct, too small to read". Los ejes, marcas, leyendas y valores de celdas de gráficos también se describen. Un ~30% de las imágenes no llevan texto; no se inventan carteles
7. **Una frase específica de iluminación**: "The lighting is …" (fuente, dirección, cualidad, sombras y altas luces)
8. **Cerrar con la composición general**: "The overall composition ⟨is/uses/feels⟩ …" **solo una frase**

**Reglas de estilo**:
- Presente, tercera persona, frases declarativas. Prohibido «you», «create», «make sure»
- **Prohibidos los quality boosters** ("masterpiece", "8K", "highly detailed", "award-winning")
- Usar atenuantes en lo incierto ("appears to be"). Solo los elementos fijados por el usuario se afirman con certeza
- **Los colores llevan modificadores** (deep navy, muted olive, pale cream). Los códigos hex solo si el usuario los especifica
- **Describir materiales** (brushed metal, matte plastic, frosted glass, weathered wood)
- Prohibido resumir con «algunos elementos»; **enumerar**. Las cantidades pequeñas se escriben en palabras (three, five, twelve)
- Las personas se describen solo por lo observable. **La edad como etapa vital, no numérica** (a young adult, in her thirties)
- Por clase, no por marca (a silver laptop, a mirrorless camera)
- Bienvenida la terminología fotográfica y de diseño (shallow depth of field, bokeh, backlit, negative space)
- Coherencia física (las sombras en dirección opuesta a la luz, reflejos consistentes)
- **El idioma de la descripción es siempre el inglés** (independientemente del idioma de la petición). Solo el texto dentro de la imagen se mantiene en su escritura original
- Salida: `{"rewritten_prompt": "<descripción>", "wh_ratio": "<ej.: 3:2>"}`

### (B) Edición (I2I) — a partir del system prompt de PE-I2I

- **Principio rector, «desentrelazado de atributos» (Attribute Disentanglement)**: editar con fuerza y claridad solo los atributos nombrados; mantener el resto con fidelidad a la imagen de entrada. Los modos de fallo son dos y simétricos: «fugas» y «sub-edición». **Mantener bloquea el contenido, no reduce la intensidad de la edición**
- **Los objetos a preservar se nombran por tipo, posición y rol; no se redescriben** (redescribir se interpreta como instrucción de generación y provoca deriva). Preferir una única cláusula de preservación inclusiva
- **La identidad es el invariante más difícil**: rostro, accesorios, diseño de producto y medio de render se mantienen en todas las ediciones salvo que se apunten explícitamente. La identidad proveniente de imágenes de referencia se señala **con etiquetas de imagen, no describiéndola con palabras**
- **El texto dentro de la imagen es literal**: si los caracteres legibles deben aparecer en la salida, se garantizan todos con exactitud y entre comillas. Prohibido omitir o resumir. No añadir texto ilegible
- **Regla obligatoria para referencias múltiples**: con N≥2, el uso de etiquetas `<image1>`, `<image2>`… es **obligatorio** (prohibidas las referencias en lenguaje natural tipo «图1» o «la primera imagen»). Con una sola imagen no se usan etiquetas. Explicitar el rol de cada imagen (lienzo / proveedor de material)
- **Decisión del idioma en dos capas**: (A) el idioma de la instrucción explicativa (instrucción en chino → chino; en inglés → inglés; en otros idiomas → inglés) y (B) **el idioma del texto dibujado en la imagen** (① especificación explícita del usuario > ② idioma dominante del texto en la imagen de entrada > ③ idioma de la instrucción), sin confundirlos. El texto dibujado debe ser **monolingüe obligatoriamente** (sin mezclas)
- **Tamaño de salida**: `wh_ratio` y `ratio_follow` (a qué salida seguir, p. ej. "<image1>") son excluyentes. Por defecto se sigue el ratio de la imagen de entrada. Solo la «generación de escena nueva» se elige semánticamente. Tabla de palabras clave de ratio: 正方形/头像→1:1, 横版/PPT→16:9, 海报→2:3, 证件照/小红书→3:4, 全景→2:1, 名片→9:5, A4→5:7/7:5, pantalla iPhone→18:39, Android→9:20, cinemascope→21:9. **"2K/4K/8K" son descriptores de calidad; no usarlos para el ratio**
- **Formato**: un solo párrafo sin saltos de línea. Comillas dobles solo para el texto dibujado en la imagen. **No incluir ratios ni resoluciones en el texto del prompt**. Describir en positivo (en lugar de «禁止改变背景», «保持背景不变»). Con determinación (sin atenuantes)

### (C) Plantilla oficial de imágenes transparentes (RGBA)

> `This is an RGBA image with transparency. <your description>. The image has alpha channel and the background is transparent.`

### (D) Flujo de trabajo recomendado oficialmente

No escribir los prompts cortos tal cual: **pasarlos por el reescritor PE para detallarlos** y después entregarlos a QwenImage21Pipeline (recomendación del README, "Prompt Rewriting"). Mapear el `wh_ratio` de salida del reescritor a la tabla de resoluciones y generar con 40 pasos.

---

## 5. Conocimiento de la comunidad (incluye generaciones anteriores; versión de aplicación indicada)

### Estructura
- **Prosa natural recomendada; las listas de tags estilo SD y la sintaxis de pesos `(red hair:1.5)` no funcionan** (todas las versiones) → enfatizar con "with vibrant, striking red hair"
- El MMDiT **pondera la posición del token y la concreción** → poner el sujeto al principio. Orden: Sujeto → Estilo → Detalles → Composición → Iluminación (fal.ai, 2512)
- En pruebas reales, la **prosa narrativa natural** puntúa mejor que la estructura ("Subject: …"). En orden directo, 95% de sujeto claro (apiyi, 2512)
- Longitud: en pruebas reales, **1–3 frases es lo óptimo** (31 palabras > 82 palabras, y además genera más rápido). Retratos: menos de 200 palabras en inglés (herramienta oficial)

### Dibujo de texto
- **La cadena a dibujar siempre entre comillas dobles** (consenso oficial y comunitario en todas las versiones)
- Solo con comillas: 85% de precisión; subiendo CFG y pasos se llega al 96% (base 65%)
- **Mayúsculas, puntuación, saltos de línea y orientación vertical/horizontal se transcriben fielmente**, según lo escrito en el prompt
- Especificar hasta tipografía, color, tamaño y forma de presentación (neón/LED/impreso/bordado/graffiti)
- Varios bloques de texto: **describir línea a línea con posición individual**
- El texto implícito («mostrar una lista») falla. **Dar la cadena literal concreta**. Cerrar con "No other text appears in the image." para evitar contaminación
- El chino es lo que mejor dibuja (94.1) > inglés > **el japonés tiende a romperse en T2I directo**. Solución: introducir una imagen con texto negro sobre blanco y pedir en Edit que lo «trace fielmente» (Zenn, 2509)

### Edición
- Base: **frases imperativas cortas**: "Change the background to a sunset beach"
- **Objetivo de edición + objeto a preservar en conjunto**: "Replace X with Y. Keep original font, size, color, and perspective. Do not alter background."
- **No atestar: dividir en 2–3 ediciones pequeñas** y encadenarlas
- Palabras clave genéricas: Replace X with Y / Add X / Remove X / Leave everything else unchanged / Rotate to show the back

### Referencias múltiples
- Los ejemplos oficiales sitúan con **lenguaje espacial**: "The magician bear is on the left, the alchemist bear is on the right, facing each other…"
- En la práctica también funciona numerar + posición: «en la habitación de la primera imagen… con la segunda imagen…» (Zenn)
- ※ El PE-I2I de 2.1 exige las etiquetas `<image1>` (ver (B) arriba) — nueva norma oficial

### Cámara e iluminación
- "shot on Canon EOS R5, 85mm f/1.4 lens" + "professional photography, RAW format"
- "Relight the scene with a warm key light from the right and cool rim light from the back. Keep pose and background unchanged."
- Sufijo oficial de la primera generación: `, Ultra HD, 4K, cinematic composition.`

### Patrones de fallo y cómo evitarlos
| Patrón de fallo | Ver. afectada | Solución |
|---|---|---|
| Tags y sintaxis de pesos no funcionan | Todas | Enfatizar con prosa natural |
| Sujeto ambiguo | Todas | Sujeto al inicio de la frase, más concreción |
| Exceso de texto rompe prioridades | 2512 | Comprimir a 1–3 frases, ~31 palabras |
| Estilos contradictorios | 2512 | Un solo estilo principal |
| Palabras vagas ("beautiful", "a list") | Todas | Sustituir por valores y cadenas concretas |
| Errores y caracteres faltantes | Todas | Comillas + CFG 6–8 + pasos 35–50. Simplificar dígitos y símbolos. Chino > inglés > japonés en estabilidad |
| Texto japonés roto | Edit-2509 | No dibujarlo directo en T2I: trazar una imagen de texto con Edit «fielmente» |
| Manos y dedos defectuosos | Todas | Negativo "extra fingers, deformed hands" + positivo "natural hand posture, five fingers" (60%→85%) |
| La edición reescribe todo | Serie Edit | Añadir siempre "Keep everything else unchanged"; una edición por instrucción |
| Rostro/identidad se degrada | Serie Edit | Indicar explícitamente "Preserve face/clothing features" |
| Se ignora el negativo | Recetas destiladas | No compatible con guidance 1; usarlo con una implementación true_cfg>1 |
| Salida ruidosa, pose rígida | 2.1 | Reportes de justo tras el lanzamiento; sin solución establecida (en observación) |

---

## 6. Prompts de ejemplo (con fuente)

### Oficial Qwen-Image-2.1 (2026-09-20)
- Dibujo de texto: `A neon shop sign that reads "QWEN IMAGE 2.1", rainy night, reflections on wet pavement`
- Edición: `Change the background to a sunset beach`
- Edición (movimiento): `Let this mascot dance under the moon`
- Referencias múltiples: `These three characters are sitting around a campfire in a forest`
- Transparencia: `This is an RGBA image with transparency. A cute cartoon dragon sticker. The image has alpha channel and the background is transparent.`
- T2I: `A capybara reading a book by candlelight` / `A ceramic teapot on a wooden table`
- Comfy oficial: `Clean flat vector infographic titled "FROM CHERRY TO CUP" showing five numbered steps left to right`

### Qwen-Image oficial de primera generación (mina de oro para texto)
- Escaparate de librería: `Bookstore window display. A sign displays "New Arrivals This Week". Below, a shelf tag with the text "Best-Selling Novels Here". To the side, a colorful poster advertises "Author Meet And Greet on Saturday" with a central portrait of the author. There are four books on the bookshelf, namely "The light between worlds" "When stars are scattered" "The slient patient" "The night circus"`
- Póster de película: `A movie poster. The first row is the movie title, which reads "Imagination Unleashed". The second row is the movie subtitle, which reads "Enter a world beyond your imagination". The third row reads "Cast: Qwen-Image". … At the bottom edge, the text "Launching in the Cloud, August 2025" appears in bold, modern sans-serif font …`
- Encerado + neón + π: `A coffee shop entrance features a chalkboard sign reading "Qwen Coffee 😊 $2 per cup," with a neon light beside it displaying "通义千问". Next to it hangs a poster showing a beautiful Chinese woman, and beneath the poster is written "π≈3.1415926-53589793-23846264-33832795-02384197".`

### Ejemplo oficial 2512 (con negativo)
- prompt: `A 20-year-old East Asian girl with delicate, charming features and large, bright brown eyes—expressive and lively... She stands indoors at an anime convention, surrounded by banners, posters, or stalls. Lighting is typical indoor illumination—no staged lighting—and the image resembles a casual iPhone snapshot...`
- negative: `低分辨率，低画质，肢体畸形，手指畸形，画面过饱和，蜡像感，人脸无细节，过度光滑，画面具有AI感。构图混乱。文字模糊，扭曲。`

### Comunidad (playbook de Reddit, 2025-08-27)
- Sustituir texto: `Replace the sign text with 'GRAND OPENING'. Keep original font, size, color, and perspective. Do not alter background or signboard.`
- Transferencia de estilo: `Re-render this scene in a Studio Ghibli art style. Preserve character identity, clothing, and layout.`
- Edición en zona marcada en rojo: `Within the red box, replace the lower component of the character '稽' with '旨'. Match stroke thickness and calligraphy style. Leave everything else unchanged.`
- Iluminación: `Relight the scene with a warm key light from the right and cool rim light from the back. Keep pose and background unchanged.`
- Lente: `Render with a 35 mm lens, shallow depth of field, focus on subject's face. Preserve environment blur.`
- Traslado con identidad preservada: `Place the same character in a desert environment. Keep hairstyle, clothing, and facial features identical.`

### Texto japonés (Zenn, 2025-10-01, Edit-2509)
- Conversión a caligrafía: `画像に書かれたテキストを習字風のフォントに変換してください。1画ごとの配置を忠実になぞり、抜け漏れがないようにしてください。左下に、赤い四角形の「通义千问」という印をつけてください`
- Sustitución de cuadro (ejemplo con referencia numerada): `1枚目の部屋に飾られている絵について、額縁は残して、その内部を2枚目の画像で表す文字に置き換えてください。フォントは入力されたゴシック体ではなく、習字のような行書体に変更してください。最後に、赤い四角のハンコを絵の左下端に加えてください`

---

## 7. Lista de fuentes

### Oficiales
| Tipo | URL | Fecha |
|---|---|---|
| Blog oficial (zh) | https://qwen.ai/blog?id=qwen-image-2.1 | 2026-09-20 |
| GitHub | https://github.com/QwenLM/Qwen-Image-2.1 | 2026-09-20 |
| Model card HF | https://huggingface.co/Qwen/Qwen-Image-2.1 | 2026-09-20 |
| PE-T2I (incluye system_prompt.txt) | https://huggingface.co/Qwen/Qwen-Image-2.1-PE-T2I | 2026-09-20 |
| PE-I2I (incluye system_prompt.txt) | https://huggingface.co/Qwen/Qwen-Image-2.1-PE-I2I | 2026-09-20 |
| Demo HF | https://huggingface.co/spaces/Qwen/Qwen-Image-2.1 | — |
| ComfyUI oficial | https://blog.comfy.org/p/qwen-image-21-in-comfyui-open-weight | 2026-09-20 |
| PR de diffusers (true_cfg) | https://github.com/huggingface/diffusers/pull/14804 | 2026-09-20 |
| Receta vLLM | https://recipes.vllm.ai/Qwen/Qwen-Image-2.1 | — |
| Cookbook de SGLang | https://docs.sglang.io/cookbook/diffusion/Qwen-Image/Qwen-Image-2.1 | — |
| Blog Qwen-Image 1.ª gen | https://qwenlm.github.io/blog/qwen-image/ | 2025-08-04 |
| README QwenLM/Qwen-Image + herramienta oficial prompt_utils_2512.py | https://github.com/QwenLM/Qwen-Image | 2025-08~2026-02 |
| Edit-2509 HF | https://huggingface.co/Qwen/Qwen-Image-Edit-2509 | 2025-09-22 |
| Edit-2511 HF | https://huggingface.co/Qwen/Qwen-Image-Edit-2511 | 2025-12-23 |

### Comunidad
| Tipo | URL | Fecha |
|---|---|---|
| Playbook de Reddit | https://www.reddit.com/r/StableDiffusion/comments/1n1n81o/ | 2025-08-27 |
| Primeras pruebas de 2.1 en Reddit | https://www.reddit.com/r/StableDiffusion/comments/1wkvqlj/ | 2026-09-19 |
| Guía 2512 de fal.ai | https://fal.ai/learn/devs/qwen-image-2512-text-to-image-prompt-guide | 2026-01-07 |
| apiyi: 23 casos de prueba | https://help.apiyi.com/en/qwen-image-2512-prompt-guide-test-cases-en.html | 2026-01-18 |
| Zenn: dibujo de japonés | https://zenn.dev/kota_iizuka/articles/33219ebb8aff99 | 2025-10-01 |
| Zenn: guía avanzada | https://zenn.dev/rick_lyric/articles/ffd10bbb59e8b6 | 2025-11-22 |

---

## 8. Implicaciones para el diseño de la skill

1. **El núcleo es la adaptación de los system_prompt.txt de PE-T2I/PE-I2I** — contienen toda la especificación oficial del «prompt ideal». Pero hay directrices opuestas entre los dos modos (T2I = descripción observacional larga de 400–500 palabras vs. pruebas comunitarias = 1–3 frases óptimas; Edición = imperativos cortos). La skill debe «elegir según el uso»
2. **SKILL.md fino, solo enrutado + principios comunes**, con el detalle dividido en references/ (progressive disclosure)
3. Contenidos imprescindibles: dibujo de texto (convención de comillas, cuidado con el japonés), cláusula de preservación en edición, convención de etiquetas `<image1>` para referencias múltiples, plantilla RGBA y tabla de ratios de aspecto
4. El prompt de salida se **genera en inglés** (especificación oficial). El texto explicativo de la skill puede estar en otro idioma
5. Descargo: el README debe indicar que la licencia es no comercial (Qwen Research License)