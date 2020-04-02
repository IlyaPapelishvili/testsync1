---
title: Async data in components?
description: Async data in NuxtJS components?
---

Because components do not have an `asyncData` method, you cannot directly fetch async data server side within a component. In order to get around this limitation you have two basic options:

1. Realice la llamada API en el gancho `montado` y establezca las propiedades de los datos cuando se cargue. * Desventaja: no funcionará para la representación del lado del servidor. *
2. Realice la llamada a la API en los métodos `asyncData` o` fetch` del componente de página y pase los datos como accesorios a los subcomponentes. La representación del servidor funcionará bien. * Desventaja: el `asyncData` o` fetch` de la página puede ser menos legible porque está cargando los datos para otros componentes *.
