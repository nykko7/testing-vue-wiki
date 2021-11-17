A pesar de que la mayoría de las veces utilizamos propiedades de tipo `computed` para reaccionar a cambios, a veces tenemos que utilizar `watchers` para poder realizar una operación, un cambio, una llamada a una API, etc. En esta entrada veremos como testear watchers y que se debe y que no se debe de hacer a la hora de testear esta herramienta.

A continuación tenemos el siguiente watcher: 

```javascript
  watch: {
    isAuthenticated(value) {
      if (value) {
        this.loading = false
        this.$router.push({ path: '/new' })
      }
    },
  },
```

Este al observar que hay un cambio en la función `isAuthenticated`, revisa si su valor es verdadero, y en caso de que si lo sea, cambiamos de ruta con ayuda del modulo de `$router`. Queremos comprobar que a la hora de hacer este cambio, este watcher si nos cambie la ruta de nuestra aplicación.

> Para ver como se hacen tests con $router, entra [aqui](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Vue/Usando-router-para-revisar-el-routing-de-nuestra-app)

Para nosotros hacer una llamada a nuestro watcher, utilizamos el método `$options.watch` que nos permite hacer una llamada a nuestro `watcher`. En nuestro caso, esto se ve de la siguiente forma:

```javascript
    wrapper.vm.$options.watch.isAuthenticated.call(wrapper.vm, wrapper.vm.isAuthenticated)
```

Vue adjunta al objeto `$options.watch` a cada uno de nuestros observadores que se definen en nuestro componente. Entonces con el método de `call` lo llamamos directamente. Su primer argumento es la instancia de nuestro componente, y su segundo argumento es el valor con el cual sera llamado. 

Con esto, podemos ver si nuestro watcher si esta realizando correctamente lo que debe dependiendo del valor que se le de. En nuestro caso, la variable `wrapper.vm.isAuthenticated` sera `true` para comprobar que la ruta si es cambiada correctamente. Para que esta tome el valor de verdadero, utilizamos la mutacion de nuestra tienda: 

```javascript
    store.commit('user/ON_AUTH_STATE_CHANGED_MUTATION', {
      authUser: {
        displayName: 'Navs',
        uid: '',
      },
    })
```

Haciendo que esta variable tome el valor de `true`. Con esto ahora solo nos falta comprobar que si haya cambiado la ruta al ejectuar nuestro `watcher`. Esto se ve de la siguiente forma: 

```javascript
expect(router.currentRoute.path).toBe('/new')
```

Con esto, pudimos acceder y probar nuestro observador de la forma correcta. Nuestro test completo se ve de la siguiente forma:

```javascript
  it('Check that when you login, you are sent to the new page', () => {
    store.commit('user/ON_AUTH_STATE_CHANGED_MUTATION', {
      authUser: {
        displayName: 'Navs',
        uid: '',
      },
    })

    wrapper = mount(Login, {
      localVue,
      vuetify,
      router,
      store,

      stubs: {
        NuxtLink: true,
      },

      mocks: {
        $t: (msg) => msg,
      },
    })

    expect(wrapper.vm.isAuthenticated).toBe(true)

    wrapper.vm.$options.watch.isAuthenticated.call(
      wrapper.vm,
      wrapper.vm.isAuthenticated
    )

    expect(router.currentRoute.path).toBe('/new')
  })
```

Ahora bien, normalmente uno podría pensar que para testar un observador, cambiamos nuestros datos y ver que nuestro `watcher` haya sido llamado. Sin embargo, esto ya es testeado por Vue, asi que no hay necesidad de realizar ese tipo de tests. Es mejor testear que la logica interna de nuestro `watcher` funcione en vez de que este es ejectuado de la forma correcta.

### Referencias
[Testing logic inside a Vue.js watcher
](https://vuedose.tips/testing-logic-inside-a-vue-js-watcher/)