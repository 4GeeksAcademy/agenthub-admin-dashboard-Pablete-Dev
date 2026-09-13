# SPECS.md — AgentHub Admin Dashboard

## 1. Descripción del producto

AgentHub es una plataforma SaaS donde empresas alquilan agentes de inteligencia artificial configurables, dotados de "skills" (habilidades/capacidades específicas) que pueden combinarse según las necesidades del cliente. Las empresas contratan uno o varios agentes, personalizan sus skills y pagan según el uso y las skills contratadas.

El entregable de este reto es el **panel de administración interno** de AgentHub: una herramienta de uso interno para el equipo de operaciones/soporte que permite:
- Visualizar el estado general del negocio (dashboard con métricas clave).
- Gestionar usuarios/clientes de la plataforma.
- Gestionar los agentes IA disponibles y su configuración.
- Gestionar el catálogo de skills.
- Consultar y administrar las contrataciones (contratos) de agentes por parte de clientes.
- Revisar un log de errores técnicos generados por los agentes en producción.

Este panel es un **prototipo estático de front-end**, sin conexión a datos reales.

## 2. Stack y restricciones

- HTML semántico (uso correcto de `<header>`, `<nav>`, `<main>`, `<section>`, `<table>`, `<dialog>`/`<div role="dialog">`, etc.).
- Tailwind CSS cargado vía CDN (script `<script src="https://cdn.tailwindcss.com"></script>` o equivalente), sin proceso de compilación.
- JavaScript vanilla (sin frameworks ni librerías de terceros).
- Prohibido usar React, Vue, jQuery, Alpine, Svelte u otro framework/librería de UI.
- Sin herramientas de build (sin bundlers, sin npm scripts de compilación, sin transpiladores).
- Sin backend ni API: no hay peticiones `fetch`/`XHR` a servidores externos ni endpoints propios.
- Todos los datos están **hardcodeados** directamente en el HTML o en constantes JavaScript embebidas.
- Diseño **responsive** para resoluciones de escritorio (≥1280px) y tablet (≥768px). No es requisito soportar móviles pequeños (<768px), aunque no debe romperse visualmente de forma catastrófica.
- Soporte de **modo claro/oscuro** implementado exclusivamente con las utilidades `dark:` de Tailwind (estrategia `class` en `tailwind.config`), alternable mediante un control en el header.

## 3. Estructura general

La aplicación es una **single-page app estática** (un único archivo HTML, o un HTML principal con secciones), compuesta por:

- **Sidebar persistente**: fijo a la izquierda, visible en todas las secciones, contiene el logo/nombre de la plataforma y la navegación principal.
- **Header / barra superior**: fijo en la parte superior del área de contenido, contiene el título de la sección activa y el control de cambio de tema.
- **Área main**: región de contenido principal donde se renderiza la sección activa.
- **Navegación por secciones** (sin recarga de página, cambiando visibilidad de bloques vía JavaScript):
  1. Dashboard
  2. Gestión de usuarios
  3. Gestión de agentes
  4. Skills
  5. Contrataciones de agentes
  6. Log de errores

Solo una sección está visible a la vez; el resto permanece oculta (`hidden`) en el DOM. La navegación de la sidebar resalta la sección activa.

## 4. Especificación por sección

### 4.1 Dashboard

1. **Tarjetas de métricas**: se muestran 4 `MetricCard` en una cuadrícula 2x2 responsive (grid de 2 columnas x 2 filas en escritorio y en tablet), cada una con icono (SVG inline o emoji), etiqueta descriptiva y valor hardcodeado:
   - Ingresos totales (ej. "€48,320").
   - Pérdidas por descuentos (ej. "€3,150").
   - Agentes activos (ej. "27").
   - Agentes fallando (ej. "4").
2. **Área de actividad semanal**: debajo de las tarjetas, un bloque full-width con borde discontinuo (`border-dashed`), altura considerable (ej. `h-64`), y una etiqueta centrada tanto vertical como horizontalmente (ej. "Gráfico de actividad semanal — próximamente") a modo de placeholder de gráfico, sin librería de charts.
3. **Comportamiento**: las tarjetas no son interactivas (no tienen acción de click); son puramente informativas. El placeholder de actividad tampoco tiene interacción. La sección se muestra por defecto al cargar la aplicación.
4. **Estados**: no aplica hover/click especial en las tarjetas más allá de un sutil `hover:shadow` opcional. No hay estados de carga (los datos ya están presentes).

### 4.2 Gestión de usuarios

1. **Estructura de tabla**: `DataTable` con columnas: Nombre, Email, Plan, Estado (badge), Acciones. Contiene al menos 5 usuarios hardcodeados con datos realistas (nombre completo, email válido en formato, plan entre "Free", "Pro", "Enterprise", estado entre "Activo", "Suspendido", "Pendiente").
2. **Badge de estado**: cada estado se muestra con `StatusBadge` de color distinto (verde=Activo, rojo=Suspendido, amarillo=Pendiente).
3. **Acciones por fila**: botón `⋮` (icono de tres puntos) que abre un `ActionDropdown` con dos opciones: "Ver detalle" y "Eliminar".
4. **Ver detalle**: abre un `Modal` con la información completa del usuario (nombre, email, plan, estado, fecha de alta hardcodeada, agentes contratados si aplica).
5. **Eliminar**: al hacer click, elimina visualmente la fila de la tabla (manipulación del DOM en el cliente, sin confirmación obligatoria pero se recomienda un `confirm()` simple o modal de confirmación).
6. **Cierre del modal**: el modal se cierra mediante un botón explícito de cierre (✕) y también al hacer click en el backdrop (fondo semitransparente detrás del modal).

### 4.3 Gestión de agentes

1. **Listado de agentes**: se muestran al menos 4 agentes, cada uno como tarjeta o fila, mostrando nombre, propietario (empresa cliente), estado (badge: "Activo", "Inactivo", "Con errores") y una lista de skills colapsada por defecto.
2. **Colapsable de skills** (`CollapsibleSkillList`): cada agente tiene un botón "Ver skills" / "Ocultar skills" que expande/colapsa la lista de skills asociadas mediante una transición CSS visible (ej. `transition-all duration-300` sobre `max-height` u `opacity`).
3. **Acciones por agente**: dropdown `⋮` con opciones "Configurar" y "Eliminar".
4. **Configurar**: abre un `Modal` que contiene un `<textarea>` editable con el prompt de sistema del agente (texto hardcodeado, editable en el cliente pero sin persistencia real al recargar).
5. **Eliminar**: elimina visualmente el agente de la lista.
6. **Estados interactivos**: el botón de expandir/colapsar cambia su texto/icono según el estado (ej. flecha que rota 180°); el dropdown se cierra al seleccionar una opción o al hacer click fuera.

### 4.4 Skills

1. **Listado de skills**: mínimo 4 skills, cada una mostrando nombre, descripción breve y cantidad de agentes que la usan (número hardcodeado).
2. **Explicación conceptual**: en la parte superior de la sección, un bloque de texto explicando qué es una skill en AgentHub (ej. "Una skill es una capacidad modular que un agente IA puede activar, como 'Atención al cliente', 'Análisis de datos' o 'Generación de reportes'. Cada agente puede combinar varias skills según el plan contratado.").
3. **Acciones por skill**: dropdown `⋮` con "Ver detalle" y "Eliminar".
4. **Ver detalle**: abre `Modal` con nombre completo, descripción extendida, lista de agentes que la usan y precio individual de la skill (dato hardcodeado).
5. **Eliminar**: elimina visualmente la skill de la lista.

### 4.5 Contrataciones de agentes

1. **Estructura de tabla**: mínimo 4 contratos, con columnas: Cliente, Agente, Skills contratadas, Fecha de inicio, Fecha de fin (o "Activo"), Importe total.
2. **Acciones por fila**: dropdown `⋮` con "Ver detalle" y "Eliminar" (o "Cancelar contrato").
3. **Ver detalle**: abre `Modal` con desglose completo del contrato: cliente, agente, fechas, lista de skills contratadas cada una con su precio individual hardcodeado, y el importe total calculado como suma (mostrado como dato ya hardcodeado, no calculado dinámicamente si no se requiere).
4. **Estados**: los contratos pueden mostrar un badge de estado ("Activo", "Finalizado", "Cancelado") junto a las fechas.
5. **Eliminar**: elimina visualmente la fila de la tabla.

### 4.6 Log de errores

1. **Estructura de tabla/lista**: mínimo 6 errores hardcodeados, cada uno con timestamp (fecha y hora), agente asociado, tipo de error (ej. "Timeout", "API Error", "Validación"), badge de gravedad/tipo (colores: rojo=crítico, naranja=advertencia, azul=informativo) y descripción breve.
2. **Acciones por fila**: dropdown `⋮` con "Ver detalle" y "Marcar como resuelto".
3. **Ver detalle**: abre `Modal` con la traza completa del error (stack trace simulado en texto monoespaciado, `<pre>` o `<code>`, hardcodeado).
4. **Marcar como resuelto**: cambia visualmente el estado/badge de la fila a "Resuelto" (ej. cambia color a verde) sin eliminar la fila.
5. **Estados interactivos**: filas ya marcadas como resueltas pueden mostrarse con opacidad reducida o badge distinto para diferenciarlas visualmente de las pendientes.

## 5. Interacciones globales

1. **Toggle claro/oscuro**: un control (switch o botón con icono sol/luna) ubicado en el header permite alternar entre modo claro y oscuro, aplicando/quitando la clase `dark` en el elemento `<html>`.
2. **Persistencia del tema al navegar**: el estado del tema (claro/oscuro) se mantiene al cambiar entre secciones dentro de la misma sesión (estado en memoria vía JavaScript, opcionalmente reforzado con `localStorage`).
3. **Cierre de dropdowns al hacer click fuera**: cualquier `ActionDropdown` abierto se cierra automáticamente si el usuario hace click en cualquier otra parte del documento fuera del dropdown.
4. **Cierre de modales**: todo `Modal` se cierra mediante: (a) un botón de cierre explícito (✕) dentro del modal, y (b) un click en el backdrop (fondo oscuro detrás del modal). Opcionalmente también con tecla `Escape`.
5. **Sidebar persistente**: la sidebar permanece visible y fija en todas las secciones, sin recargar ni desmontarse al navegar.
6. **Resaltado de sección activa**: el ítem de navegación correspondiente a la sección visible se resalta visualmente (ej. fondo distinto, texto en color de acento, borde lateral) tanto en modo claro como oscuro.

## 6. Inventario de componentes reutilizables

- **Sidebar**: Propósito — navegación principal persistente. Contenido — logo/nombre "AgentHub", lista de 6 enlaces de navegación (uno por sección) con icono y etiqueta. Comportamiento — al hacer click en un enlace, cambia la sección visible en el área main y resalta el enlace activo; permanece fijo al hacer scroll.

- **Topbar/Header**: Propósito — mostrar contexto de la sección actual y controles globales. Contenido — título de la sección activa, control `ThemeToggle`. Comportamiento — se mantiene fijo en la parte superior del área de contenido; su título cambia dinámicamente según la sección activa.

- **MetricCard**: Propósito — mostrar una métrica clave de forma visual. Contenido — icono, etiqueta descriptiva, valor numérico/monetario hardcodeado. Comportamiento — puramente informativo, sin interacción de click; puede tener efecto `hover` sutil.

- **DataTable**: Propósito — listar registros tabulares (usuarios, contratos, errores). Contenido — cabecera de columnas, filas de datos, columna de acciones con `ActionDropdown`. Comportamiento — filas pueden eliminarse o actualizarse visualmente mediante las acciones; scroll horizontal en pantallas estrechas si es necesario.

- **StatusBadge**: Propósito — indicar visualmente un estado o gravedad. Contenido — texto corto (ej. "Activo", "Crítico") con color de fondo/texto según el tipo. Comportamiento — puramente visual, sin interacción; los colores deben tener contraste adecuado en modo claro y oscuro.

- **ActionDropdown**: Propósito — agrupar acciones contextuales por fila/tarjeta. Contenido — botón disparador `⋮` y menú desplegable con 2 opciones de texto. Comportamiento — se abre/cierra al hacer click en el botón; se cierra al hacer click fuera o al seleccionar una opción; solo un dropdown puede estar abierto a la vez.

- **Modal**: Propósito — mostrar información detallada o formularios sin abandonar la vista actual. Contenido — backdrop semitransparente, contenedor centrado con título, contenido específico (detalle, formulario, traza), botón de cierre (✕). Comportamiento — se abre mediante una acción del usuario (ej. "Ver detalle"), se cierra con el botón ✕ o click en el backdrop; bloquea la interacción con el contenido de fondo mientras está abierto.

- **CollapsibleSkillList**: Propósito — mostrar/ocultar una lista de skills asociadas a un agente para ahorrar espacio. Contenido — lista de nombres de skills (chips o lista simple), botón/indicador de expandir-colapsar. Comportamiento — alterna entre estado colapsado y expandido con una transición CSS visible; el icono/flecha indicador rota o cambia según el estado.

- **ThemeToggle**: Propósito — permitir alternar entre modo claro y oscuro. Contenido — botón o switch con icono de sol/luna. Comportamiento — al activarse, añade/quita la clase `dark` en `<html>`; refleja visualmente el estado actual (icono activo).

- **Empty/PlaceholderChart**: Propósito — representar visualmente un espacio reservado para un gráfico futuro sin implementar una librería de charts. Contenido — contenedor con borde discontinuo y etiqueta centrada. Comportamiento — puramente visual, sin datos ni interacción.

- **SectionHeader**: Propósito — encabezar cada sección de contenido con un título y, opcionalmente, una breve descripción o acción contextual. Contenido — título (ej. "Gestión de usuarios"), subtítulo o descripción corta opcional. Comportamiento — estático, cambia su contenido según la sección activa.

## 7. Criterios de aceptación

1. La aplicación carga con la sección **Dashboard** visible por defecto.
2. La sidebar es visible y persistente en las 6 secciones, sin recargar la página al navegar entre ellas.
3. El header muestra el título correspondiente a la sección activa en todo momento.
4. El ítem de navegación de la sección activa se muestra visualmente resaltado respecto a los demás.
5. El Dashboard muestra exactamente 4 `MetricCard` con icono, etiqueta y valor hardcodeado (ingresos totales, pérdidas por descuentos, agentes activos, agentes fallando).
6. Debajo de las métricas, el Dashboard muestra un bloque full-width con borde discontinuo y una etiqueta centrada simulando un área de actividad semanal.
7. La sección de Gestión de usuarios contiene una tabla con al menos 5 usuarios, cada uno con nombre, email, plan y badge de estado.
8. Cada fila de usuario tiene un dropdown `⋮` funcional con las opciones "Ver detalle" y "Eliminar".
9. "Ver detalle" en usuarios abre un modal con la información completa del usuario seleccionado.
10. La sección de Gestión de agentes contiene al menos 4 agentes con nombre, propietario, estado y lista de skills colapsada por defecto.
11. Cada agente permite expandir y colapsar su lista de skills mediante una transición CSS visible (no un cambio instantáneo).
12. Cada agente tiene un dropdown `⋮` con "Configurar" y "Eliminar"; "Configurar" abre un modal con un `<textarea>` editable que contiene el prompt de sistema.
13. La sección de Skills contiene al menos 4 skills con nombre, descripción y número de agentes que la usan, además de un texto explicativo de qué es una skill en AgentHub.
14. Cada skill tiene un dropdown `⋮` con "Ver detalle" (abre modal) y "Eliminar".
15. La sección de Contrataciones contiene al menos 4 contratos con cliente, agente, skills, fechas e importe.
16. "Ver detalle" en contrataciones abre un modal con el desglose del contrato, incluyendo el precio individual de cada skill contratada.
17. La sección de Log de errores contiene al menos 6 errores con timestamp, agente, tipo, badge de gravedad/tipo y descripción.
18. Cada error tiene un dropdown `⋮` con "Ver detalle" (abre modal con traza completa) y "Marcar como resuelto" (actualiza el estado visual de la fila sin eliminarla).
19. En todas las secciones, únicamente un `ActionDropdown` puede estar abierto a la vez, y se cierra automáticamente al hacer click fuera de él.
20. Todo `Modal` se puede cerrar tanto con su botón de cierre (✕) como haciendo click en el backdrop.
21. El control `ThemeToggle` del header alterna correctamente entre modo claro y oscuro, aplicando las utilidades `dark:` de Tailwind en todos los componentes visibles.
22. El estado del tema (claro/oscuro) seleccionado se mantiene al cambiar de sección dentro de la misma sesión.
23. El layout es utilizable y visualmente correcto tanto en resoluciones de escritorio (≥1280px) como de tablet (≥768px), sin overlaps ni contenido cortado.
24. No existe ninguna llamada a red (`fetch`/`XHR`) ni dependencia de backend; todos los datos están hardcodeados en el HTML/JavaScript.
25. No se utiliza ningún framework o librería de UI (React, Vue, jQuery, etc.) ni herramienta de build; solo HTML, Tailwind vía CDN y JavaScript vanilla.
