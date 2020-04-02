---
title: Datos asincrónicos
description: You may want to fetch data and render it on the server-side. Nuxt.js adds an `asyncData` method to let you handle async operations before setting the component data.
---

> Es posible que desee obtener datos y representarlos en el lado del servidor. Nuxt.js agrega un método `asyncData` para permitirle manejar operaciones asincrónicas antes de inicializar el componente

<div>
  <a href="https://vueschool.io/courses/async-data-with-nuxtjs?friend=nuxt" target="_blank" class="Promote"></a><a href="https://vueschool.io/courses/async-data-with-nuxtjs?friend=nuxt" target="_blank" class="Promote">
    <img src="/async-data-with-nuxtjs.png" srcset="/async-data-with-nuxtjs-2x.png 2x" alt="AsyncData by vueschool">
    <div class="Promote__Content">
      <h4 class="Promote__Content__Title">Async Data with Nuxt.js</h4>
      <p class="Promote__Content__Description">Learn how to manage asynchronous data with Nuxt.js.</p>
      <p class="Promote__Content__Signature">Video courses made by VueSchool to support Nuxt.js development.</p>
    </div>
  </a><a href="https://vueschool.io/courses/async-data-with-nuxtjs?friend=nuxt" target="_blank" class="Promote">
    <img src="/async-data-with-nuxtjs.png" srcset="/async-data-with-nuxtjs-2x.png 2x" alt="AsyncData by vueschool">
    <div class="Promote__Content">
      <h4 class="Promote__Content__Title">Async Data with Nuxt.js</h4>
      <p class="Promote__Content__Description">Learn how to manage asynchronous data with Nuxt.js.</p>
      <p class="Promote__Content__Signature">Video courses made by VueSchool to support Nuxt.js development.</p>
    </div>
  </a>
</div>

## El método asyncData

A veces solo desea obtener datos y renderizarlos previamente en el servidor sin usar una tienda.
Se llama a `asyncData` cada vez antes de cargar el componente de **página** .
Se llamará del lado del servidor una vez (en la primera solicitud a la aplicación Nuxt) y del lado del cliente cuando navegue a otras rutas.
Este método recibe [el contexto](/api/context) como primer argumento, puede usarlo para obtener algunos datos y Nuxt.js lo fusionará con los datos del componente.

Nuxt.js fusionará automáticamente el objeto devuelto con los datos del componente.

<div class="Alert Alert--orange">
</div>

**NO** tiene acceso a la instancia del componente a través de `this` `asyncData` porque se llama **antes de iniciar** el componente.




Nuxt.js le ofrece diferentes formas de usar `asyncData` . Elija el que le resulte más familiar:

1. Devolviendo una `Promise` . Nuxt.js esperará a que se resuelva la promesa antes de representar el componente.
2. Uso de [async / await](https://javascript.info/async-await) ( [obtenga más información al respecto](https://zeit.co/blog/async-and-await) )

<div class="Alert Alert--grey">
</div>

Estamos utilizando [axios](https://github.com/mzabriskie/axios) para realizar solicitudes HTTP isomórficas, <strong>recomendamos encarecidamente</strong> utilizar nuestro [módulo axios](https://axios.nuxtjs.org/) para sus proyectos Nuxt.




Si está utilizando `axios` directamente desde `node_modules` y ha utilizado `axios.interceptors` para agregar interceptores para transformar los datos, asegúrese de crear una instancia antes de agregar interceptores. De lo contrario, cuando actualice la página serverRender, los interceptores se agregarán varias veces, lo que provocará un error de datos.

```js
import axios from 'axios'
const myaxios = axios.create({
  // ...
})
myaxios.interceptors.response.use(function (response) {
  return response.data
}, function (error) {
  // ...
})
```

### Devolviendo una promesa

```js
export default {
  asyncData ({ params }) {
    return axios.get(`https://my-api/posts/${params.id}`)
      .then((res) => {
        return { title: res.data.title }
      })
  }
}
```

### Usando async / await

```js
export default {
  async asyncData ({ params }) {
    const { data } = await axios.get(`https://my-api/posts/${params.id}`)
    return { title: data.title }
  }
}
```

### Mostrar los datos

El resultado de asyncData se **fusionará** con los datos.
Puede mostrar los datos dentro de su plantilla como está acostumbrado a hacer:

```html
<template>
  <h1>{{ title }}</h1>
</template>
```

## El contexto

Para ver la lista de claves disponibles en `context` , eche un vistazo al <a href="/api/context" data-md-type="link">`context` API Essential</a> .

### Usar objetos `req` / `res`

Cuando se llama a `asyncData` en el lado del servidor, tiene acceso a los objetos `req` y `res` de la solicitud del usuario.

```js
export default {
  async asyncData ({ req, res }) {
    // Please check if you are on the server side before
    // using req and res
    if (process.server) {
      return { host: req.headers.host }
    }

    return {}
  }
}
```

### Acceso a datos de ruta dinámica

¡También puede usar el parámetro de `context` para acceder a los datos de la ruta dinámica!
Por ejemplo, los parámetros de ruta dinámica se pueden recuperar utilizando el nombre del archivo o carpeta que lo configuró.
Si ha definido un archivo llamado `_slug.vue` en su carpeta de `pages` , puede acceder al valor a través de `context.params.slug` :

```js
export default {
  async asyncData ({ params }) {
    const slug = params.slug // When calling /abc the slug will be "abc"
    return { slug }
  }
}
```

### Escuchando los cambios de consulta

The `asyncData` method **is not called** on query string changes by default.
If you want to change this behavior, for example when building a pagination component,
you can set up parameters that should be listened to with the `watchQuery` property of your page component.
Learn more on the <a href="/api/pages-watchquery" data-md-type="link">API `watchQuery` page</a> page.

## Errores de manejo

Nuxt.js agrega el método de `error(params)` en el `context` , al que puede llamar para mostrar la página de error. `params.statusCode` también se usará para representar el código de estado adecuado desde el lado del servidor.

Ejemplo con una `Promise` :

```js
export default {
  asyncData ({ params, error }) {
    return axios.get(`https://my-api/posts/${params.id}`)
      .then((res) => {
        return { title: res.data.title }
      })
      .catch((e) => {
        error({ statusCode: 404, message: 'Post not found' })
      })
  }
}
```

Para personalizar la página de error, eche un vistazo a la [guía de vistas](/guide/views#layouts) .
