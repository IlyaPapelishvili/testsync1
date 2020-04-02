---
title: ¿Datos asíncronos en componentes?
description: ¿Datos asíncronos en componentes NuxtJS?
---

Debido a que los componentes no tienen un método `asyncData` , no puede buscar directamente el lado del servidor de datos asíncrono dentro de un componente. Para evitar esta limitación, tiene dos opciones básicas:

1. Realice la llamada API en el gancho `mounted` y establezca las propiedades de los datos cuando se cargue. *Desventaja: no funcionará para la representación del lado del servidor.*
2. Realizar la llamada API en el `asyncData` o `fetch` métodos del componente página y pasar los datos como apoyos a los subcomponentes. La representación del servidor funcionará bien. *Desventaja: el `asyncData` o la `fetch` de la página pueden ser menos legibles porque está cargando los datos para otros componentes* .
