## Overview

### 1. El mapa de capas

Las herramientas de programación con IA son una pila de cuatro capas, y mucha de la confusión viene de mezclarlas:

1. **Modelos** (Claude, GPT, DeepSeek, GLM, Qwen, etc.): el "cerebro". Reciben texto y devuelven texto, y se usan por API.
2. **Agentes de código** (Claude Code, Codex, OpenCode): programas que envuelven a un modelo en un bucle de trabajo para leer archivos, editar y ejecutar comandos.
3. **Runtimes o espacios de trabajo** (herdr, Rex): organizan varios agentes en paralelo, mantienen las sesiones vivas y muestran cuál necesita tu atención. No son agentes ni modelos.
4. **La máquina o sandbox**: donde todo corre (tu laptop, un servidor, un entorno aislado).

### 2. herdr y Rex frente a tmux

Si corres todo en una sola máquina, tmux sirve perfectamente. herdr y Rex agregan conocimiento sobre los agentes: estado de cada uno (trabajando, inactivo, bloqueado), notificaciones y una API para que los agentes controlen el espacio de trabajo. Mi recomendación fue empezar con tmux y pasar a esas herramientas solo si te encuentras revisando paneles a mano.

### 3. Cómo elegir modelo

- **Open weight** significa que los parámetros del modelo se pueden descargar. No es lo mismo que open source.
- Puedes usarlos por la API del creador, por un proveedor tercero o gateway, o en local (que exige hardware potente para los modelos grandes).
- Las familias principales son DeepSeek, GLM, Kimi, Qwen y MiniMax, con Mistral y Meta como novedades.
- Los benchmarks sirven para armar una lista corta, no para decidir, porque muchos los reporta el propio fabricante.
- El método: elegir 2 o 3 candidatos, revisar la licencia, comparar el costo total (no solo el precio por token) y probarlos con tus propias tareas.
- Para presupuesto limitado: un modelo open weight barato por defecto y uno caro solo para los problemas difíciles.

### 4. Cómo se financian los laboratorios

DeepSeek fue una excepción: lo financió el fondo cuantitativo High-Flyer hasta que, en 2026, abrió su capital a inversores externos. Los ingresos de su API crecen y tienen buen margen, pero el margen bruto no incluye el enorme costo de entrenar modelos nuevos. El patrón habitual en el resto de la industria combina rondas de capital de riesgo, inversión de grandes tecnológicas, ingresos propios y, a veces, dinero estatal. Publicar los pesos es una decisión estratégica (adopción, reputación, presión sobre competidores), no caridad. La consecuencia para ti es que los precios bajos de hoy dependen en parte de que alguien subsidie la investigación, así que conviene no atarte a un solo modelo.

### 5. Gateways y OpenCode

Un gateway es un intermediario que te da una sola cuenta y factura para acceder a muchos modelos, a cambio de que tu tráfico pase por un tercero. OpenCode es en realidad un agente (capa 2), pero la empresa que lo desarrolla, Anomaly, también vende acceso a modelos:

- **El agente** es gratuito y de código abierto.
- **Zen** es pago por uso, sin margen sobre los tokens y con una comisión por recarga.
- **Go** es una suscripción mensual barata orientada a modelos open weight.

Anomaly es una startup pequeña respaldada por Y Combinator, sin el gasto de entrenar modelos, y su negocio es revender acceso con poco margen y mucho volumen. Como advertencia, hubo una brecha de seguridad en su servicio alojado (la Console) que expuso claves de algunos usuarios. Quienes usaban el agente local con sus propias claves no fueron afectados.

### Qué te llevas

Para empezar con poco gasto: usa OpenCode con modelos open weight baratos, ya sea con tus propias claves (lo más flexible y seguro) o con Go. Empieza con tmux, y prueba cada modelo con tus propias tareas antes de comprometerte. Los precios, modelos y límites cambian cada pocas semanas, así que verifica en las páginas oficiales antes de pagar.

Los detalles y las fuentes están en mis respuestas anteriores, y la parte de OpenCode y financiamiento quedó en el documento que creamos.


## OpenCode

Los modelos open weight se pueden descargar, así que cualquier empresa con GPUs puede ejecutarlos. Existen proveedores de inferencia que alquilan ese servicio (Baseten, Together, Fireworks y otros). Según un caso de estudio publicado por Baseten, OpenCode probó varios proveedores de inferencia para modelos open source y encontró problemas: algunos rendían solo 15 tokens por segundo, tenían hasta un 10% de errores en las llamadas a herramientas y respuestas lentas. Esa fuente indica que Baseten es uno de los proveedores de inferencia principales de OpenCode para varios modelos. Ten en cuenta que ese texto lo escribió el propio proveedor como material comercial, así que sirve para entender el modelo, no como evaluación independiente. Tampoco encontré una declaración explícita de que Anomaly no tenga hardware propio, pero todo lo publicado apunta a que alquilan la inferencia.

Anomaly aporta tres cosas encima de eso:

- **Curaduría y pruebas.** Su documentación dice que probaron un grupo selecto de modelos abiertos, hablaron con sus equipos sobre cómo ejecutarlos mejor y trabajaron con algunos proveedores para asegurar que se sirvieran correctamente. Esto importa porque la calidad y disponibilidad varían entre proveedores, y el mismo modelo puede comportarse distinto según quién lo ejecute.
- **Volumen y optimización de costos.** El caso de estudio afirma que las altas tasas de caché y el precio de tokens en caché lograron una reducción de costo combinado del 33% frente a sus proveedores anteriores, y que OpenCode trasladó parte del ahorro a los usuarios de Zen.
- **Un solo punto de acceso.** Tú te conectas a un endpoint de OpenCode y ellos enrutan hacia el proveedor que sirve ese modelo.

## Por qué no es la API de DeepSeek

Esto tiene una consecuencia práctica. Si fuera la API de DeepSeek, tus datos viajarían a los servidores de ese laboratorio. En cambio, todos los modelos de Zen están alojados en Estados Unidos, es decir, un proveedor estadounidense ejecuta los pesos de DeepSeek (o de GLM, Kimi, etc.) y tu código no pasa por el laboratorio original. Ese es uno de los beneficios concretos de los modelos open weight que mencionamos al principio.

## Una distinción sobre los modelos cerrados

Zen también lista modelos de OpenAI, como GPT 5.6 Sol y Terra. Esos pesos no son públicos, así que no pueden ejecutarlos terceros por su cuenta; OpenCode tiene que pasar por la API de OpenAI. Esto es una inferencia mía a partir de que esos modelos son cerrados, porque las fuentes no detallan el acuerdo comercial. Lo mismo valdría para Claude o Gemini.

## En resumen

El ahorro viene principalmente de que los modelos open weight ya son baratos de ejecutar, de que un proveedor de inferencia compite por servirlos, y de que Anomaly negocia con el volumen agregado de sus usuarios. Tú no pagas por un "modelo de OpenCode": pagas por acceso a un modelo ajeno, servido por una infraestructura ajena, con la curaduría y la facturación de OpenCode encima.