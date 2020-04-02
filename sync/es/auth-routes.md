---
title: Rutas de autenticación
description: Ejemplo de rutas autenticadas con Nuxt.js
github: rutas de autenticación
livedemo: https://nuxt-auth-routes.gomix.me
liveedit: https://gomix.com/#!/project/nuxt-auth-routes
---

# Documentación

> Nuxt.js se puede usar para crear rutas autenticadas fácilmente.

## `auth-module` oficial

Si desea implementar flujos de autenticación complejos, por ejemplo OAuth2, le sugerimos que use el [`auth-module`](https://github.com/nuxt-community/auth-module) oficial

## Usando Express y Sesiones

Para agregar la función de sesiones en nuestra aplicación, usaremos `express` y `express-session` , para esto, necesitamos usar Nuxt.js mediante programación.

Primero, instalamos las dependencias:

```bash
yarn add express express-session body-parser whatwg-fetch
```

*Hablaremos de `whatwg-fetch` más tarde.*

Luego creamos nuestro `server.js` :

```js
const { Nuxt, Builder } = require('nuxt')
const bodyParser = require('body-parser')
const session = require('express-session')
const app = require('express')()

// Body parser, to access `req.body`
app.use(bodyParser.json())

// Sessions to create `req.session`
app.use(session({
  secret: 'super-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: { maxAge: 60000 }
}))

// POST `/api/login` to log in the user and add him to the `req.session.authUser`
app.post('/api/login', function (req, res) {
  if (req.body.username === 'demo' && req.body.password === 'demo') {
    req.session.authUser = { username: 'demo' }
    return res.json({ username: 'demo' })
  }
  res.status(401).json({ error: 'Bad credentials' })
})

// POST `/api/logout` to log out the user and remove it from the `req.session`
app.post('/api/logout', function (req, res) {
  delete req.session.authUser
  res.json({ ok: true })
})

// We instantiate Nuxt.js with the options
const isProd = process.env.NODE_ENV === 'production'
const nuxt = new Nuxt({ dev: !isProd })
// No build in production
if (!isProd) {
  const builder = new Builder(nuxt)
  builder.build()
}
app.use(nuxt.render)
app.listen(3000)
console.log('Server is listening on http://localhost:3000')
```

Y actualizamos nuestros scripts `package.json` :

```json
// ...
"scripts": {
  "dev": "node server.js",
  "build": "nuxt build",
  "start": "cross-env NODE_ENV=production node server.js"
}
// ...
```

Nota: Deberá ejecutar `npm install --save-dev cross-env` para que el ejemplo anterior funcione. Si usted *no* está desarrollando en Windows puede dejar cruzada env de su `start` guión y conjunto `NODE_ENV` directamente.

## Usando la tienda

Necesitamos un estado global para que nuestra aplicación sepa si el usuario está conectado a **través de las páginas** .

Para permitir que Nuxt.js use Vuex, creamos un archivo `store/index.js` :

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

// Polyfill for `window.fetch()`
require('whatwg-fetch')

const store = () => new Vuex.Store({

  state: () => ({
    authUser: null
  }),

  mutations: {
    SET_USER (state, user) {
      state.authUser = user
    }
  },

  actions: {
    // ...
  }

})

export default store
```

1. Importamos `Vue` y `Vuex` (incluido en Nuxt.js) y le decimos a Vue que use Vuex para permitirnos usar `$store` en nuestros componentes.
2. `require('whatwg-fetch')` para rellenar el método `fetch()` en todos los navegadores (ver [repositorio de fetch](https://github.com/github/fetch) ).
3. Creamos nuestra mutación `SET_USER` que establecerá el `state.authUser` para el usuario conectado.
4. Exportamos nuestra instancia de la tienda a Nuxt.js puede inyectarla en nuestra aplicación principal.

### acción nuxtServerInit ()

Nuxt.js llamará a una acción específica llamada `nuxtServerInit` con el contexto en argumento, por lo que cuando se cargue la aplicación, la tienda ya estará llena de algunos datos que podemos obtener del servidor.

En nuestra `store/index.js` , podemos agregar la acción `nuxtServerInit` :

```js
nuxtServerInit ({ commit }, { req }) {
  if (req.session && req.session.authUser) {
    commit('SET_USER', req.session.authUser)
  }
}
```

Para que el método de datos sea asíncrono, Nuxt.js le ofrece diferentes formas, elija la que le resulte más familiar:

1. devolviendo una `Promise` , Nuxt.js esperará a que se resuelva la promesa antes de representar el componente.
2. Uso de la [propuesta `async` / `await`](https://github.com/lukehoban/ecmascript-asyncawait) ( [obtenga más información al respecto](https://zeit.co/blog/async-and-await) ).

### acción login ()

Agregamos una acción de `login` que se llamará desde nuestro componente de páginas para iniciar sesión en el usuario:

```js
login ({ commit }, { username, password }) {
  return fetch('/api/login', {
    // Send the client cookies to the server
    credentials: 'same-origin',
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      username,
      password
    })
  })
  .then((res) => {
    if (res.status === 401) {
      throw new Error('Bad credentials')
    } else {
      return res.json()
    }
  })
  .then((authUser) => {
    commit('SET_USER', authUser)
  })
}
```

### método logout ()

```js
logout ({ commit }) {
  return fetch('/api/logout', {
    // Send the client cookies to the server
    credentials: 'same-origin',
    method: 'POST'
  })
  .then(() => {
    commit('SET_USER', null)
  })
}
```

## Componentes de páginas

Entonces podemos usar `$store.state.authUser` en los componentes de nuestras páginas para verificar si el usuario está conectado a nuestra aplicación o no.

### Redireccionar usuario si no está conectado

Agreguemos una ruta `/secret` donde solo el usuario conectado puede ver su contenido:

```html
<template>
  <div>
    <h1>Super secret page</h1>
    <router-link to="/">Back to the home page</router-link>
  </div>
</template>

<script>
export default {
  // we use fetch() because we do not need to set data to this component
  fetch ({ store, redirect }) {
    if (!store.state.authUser) {
      return redirect('/')
    }
  }
}
</script>
```

Podemos ver en el método de `fetch` que llamamos `redirect('/')` cuando nuestro usuario no está conectado.

