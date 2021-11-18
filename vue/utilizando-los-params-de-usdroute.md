Anteriormente se ha hablado de como mockear `$router` en nuestras aplicaciones y también en caso de necesitarlo, utilizar la librería de `router` en su totalidad. Ahora bien, que pasa si en un archivo queremos mockear una parte de esta libreria, pero queremos utilizar sus funcionalidades reales?

Aqui tenemos dos opciones, la primera es mockear la librería y simular los métodos, por ejemplo, si queremos utilizar el método de `push` de `$router`, tendríamos que mockearlo o simular su funcionamiento para poder utilizarlo. Sin embargo, esto puede ser muy tardado y propenso a errores. Si quieres mockear parametros de `$router` y utilizar `$router`, mejor utiliza la librería de `$router` y define parametros "mock" dentro de tu test. Veamos un ejemplo.

A continuación, tenemos el siguiente botón, el cual al hacer click, nos lleva a una página que depende de distintos parametros:

```html
      <v-btn
         id="regresar"
         color="primary"
         class="mr-2"
         :to="`/${$route.params.id}/forms/answers/${$route.params.form}`"
       >
``` 

Si utilizamos el mock de route, tenemos acceso a estos valores de `params`, sin embargo no podemos testear si una vez se haga click en el botón, si este si nos lleva a la dirección correcta, ya que no contamos con esa funcionalidad. Como podemos solucionar esto entonces? Una forma es creando rutas "mock" en nuestro test con los parametros que nosotros deseamos. En código, esto se ve de la siguiente forma:

```javascript
localVue.use(VueRouter)
const router = new VueRouter({
  routes: [
    {
      path: '/',
      name: 'home',
    },
  ],
})
```

Primeramente, usamos `localVue` para indicar que vamos a utilizar la librería de VueRouter. Una vez hecho esto, le damos como argumento a nuestro nuevo VueRouter, las rutas que vamos a tener en nuestro test. En nuestro caso, pusimos "home" que tendrá la ruta de "/". 

Con nuestras rutas ya puestas, al iniciar nuestro test, usamos `router.push` para movernos a esta ruta, pero al hacerlo, tambien damos los parametros o `params` que queremos que tenga nuestro test, para que pueda utilizarlos:

```javascript
router.push({ name: 'home', params: { id: '1', form: '22' } })
```

Con esto, nuestro test sabe que estamos en la ruta de 'home' y que tenemos los parametros de `id` y `form` disponibles. De esta forma, al hacer click en el botón, nosotros esperamos que la ruta cambie a `/1/forms/answers/22`, ya que estos datos son accesibles a nuestro test. Nuestra prueba se ve entonces de la siguiente forma:

```javascript
  it('Clicking on the "regresar" button takes you to the correct page', async () => {
    router.push({ name: 'home', params: { id: '1', form: '22' } })

    wrapper = mount(asnwID, {
      localVue,
      vuetify,
      store,
      router,

      mocks: {
        $t: (msg) => msg,
        $fire,
      },
      stubs: { 'no-ssr': true },
    })

    const backButton = wrapper.find('#regresar')
    backButton.trigger('click')
    await wrapper.vm.$nextTick()

    expect(router.currentRoute.path).toBe('/1/forms/answers/22')
  })
```

Con esto, podemos utilizar los params que deseamos y además utilizar todas las funcionalidades que nos da VueRouter en nuestro test. 

### Referencias
- [Programmatic Navigation](https://router.vuejs.org/guide/essentials/navigation.html)
- [How to use routing in Vue.js to create a better user experience](https://www.freecodecamp.org/news/how-to-use-routing-in-vue-js-to-create-a-better-user-experience-98d225bbcdd9/)