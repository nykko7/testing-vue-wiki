En otra entrada de la Wiki se habló acerca de como se puede realizar un mock de la herramienta `$route` y `$router` para poder configurar nuestros tests correctamente. Sin embargo, a la hora de mockearlos, estos tenian un valor predefinido o un valor constante que nosotros le dabamos. En esta entrada veremos como podemos utilizar la libreria de `$router` en nuestros tests sin tener que utilizar un mock y como es que se debe de configurar.

<h3> Configuracion de $router </h3>

Primero que nada tenemos que configurar nuestro test para poder utilizar router a la hora de montar nuestra aplicación. Para esto, primero tenemos que importarla al inicio de nuestro test, junto con el resto de las otras importaciones: 

```javascript
...
import VueRouter from 'vue-router'
```

Una vez importado, vamos a utilizar este modulo en nuestro `localVue`. Esto se realiza de igual forma que cuando queremos utilizar Vuex en nuestros tests:

```javascript
localVue.use(VueRouter)
const router = new VueRouter()
```

A la hora de montar nuestro componente, dentro de las opciones de montado, le vamos a dar como argumento la constante `router` que acabamos de crear. Con esto, nuestro test no solo podrá funcionar correctamente, si no que podrá aprovechar todas las funcionalidades que nos da esta herramienta: 

```javascript
    wrapper = mount(NavbarND, {
      localVue,
      store,
      vuetify,
      router,
      
      ...
    })
```

<h3> Testing $router </h3>

Ahora vamos a realizar nuestro test. En este caso, queremos probar si nosotros al cerrar sesión en nuestra aplicación, esta nos lleve a la pantalla de login. Para probar esto, primero encontramos nuestro botón de cerrar sesión y lo almacenamos en una constante. Una vez hecho esto, con ayuda del método de `trigger` hacemos que se haga click en el botón de cerrar sesión.

```javascript
    const logoutButton = wrapper.find('#logoutBtn')

    logoutButton.trigger('click')
    await wrapper.vm.$nextTick()
```

 En el componente, tenemos que una vez que se haga click en este botón, se dispare el siguiente método:

```javascript
    async exit() {
      try {
        this.logoutLoading = true
        await logOut(this.$fire)
        this.$router.push({ path: '/login' })
        this.logoutLoading = false
      } catch (error) {
        this.logoutLoading = false
        console.error(error.message)
      }
    },
```

Aqui podemos notar que tenemos un `push` a la ruta de login una vez que el usuario haga clic en el botón. Nosotros entonces esperamos que una vez se active este método, nuestra ruta cambie a `/login`. Para comprobar esto, podemos acceder a la ruta actual desde nuestro test con los métodos encadenados: `currentRoute.path `:

```javascript
expect(router.currentRoute.path).toBe('/login')
```

Con esto, nuestro test queda completo y verificamos que `$router` funciona correctamente y que nuestra aplicación si esta redireccionando correctamente a nuestro usuario. 

```javascript
  it('Validate that when loging out, you are sent to the login page', async () => {
    wrapper = mount(NavbarND, {
      localVue,
      store,
      vuetify,
      router,
      mocks: {
        $t: (msg) => msg,
        $fire,
      },
      stubs: ['router-link'],
    })

    const logoutButton = wrapper.find('#logoutBtn')

    logoutButton.trigger('click')
    await wrapper.vm.$nextTick()

    expect(router.currentRoute.path).toBe('/login')

    expect($fire.auth.signOut).toHaveBeenCalled()
  })
```



<h3> Referencias </h3>

- [Vue-Test-Utils Unknown custom element: <router-link>](https://stackoverflow.com/questions/49681546/vue-test-utils-unknown-custom-element-router-link)

- [Vue test-utils how to test a router.push()](https://stackoverflow.com/questions/53302536/vue-test-utils-how-to-test-a-router-push)

- [Testing Vue Router](https://next.vue-test-utils.vuejs.org/guide/advanced/vue-router.html)