Al usar Nuxt en nuestra aplicación, puede que utilicemos los hooks como `asyncData` o `fetch` para realizar ciertas acciones como recolección de datos dentro de nuestro componente. Estos aportan código a nuestra aplicación, por lo tanto, estos también forman parte de nuestro test coverage. Como podemos testear estos hooks ajenos a Vue?

Los hooks de Nuxt son hooks de un API tercero, el cual depende totalmente en como funciona Nuxt, el cual le da la funcionalidad de Nuxt al componente o página que estamos realizando. vue-test-utils no soporta hacer tests para librerias de terceros, sin embargo, sabemos que estas se pueden mockear utilizando espias. El problema es que si tratamos de encontrar la función de `fetch` en nuestro wrapper, no lo vamos a encontrar, ya que fetch() se ejecuta antes de que el componente se monta. 

```javascript
  async fetch() {
    const self = this
    let id = null
    if (this.formId !== '') {
      id = self.formId
    } else {
      id = this.$route.params.builderid
    }
    await this.$fire.firestore
      .collection(`projects/${this.$route.params.id}/forms`)
      .doc(id)
      .get()
      .then((doc) => {
        const { name, items } = doc.data()
        self.list = items
        self.name = name
      })
  },
```

Para poder acceder nosotros a esta función (o cualquier función de una API de terceros), utilizamos el método de `$options`, en donde se "esconde" nuestra función de Nuxt.

```javascript
const spyFetch = jest.spyOn(wrapper.vm.$options, 'fetch')
```

Ahora bien, si tratamos de llamar a nuestro método como anteriormente se ha hecho (llamandolo directamente con wrapper.vm) esto nos arroja un error porque no reconoce nuestro mock de `firebase`. Esto se debe a que el mock de firebase esta en el contexto de wrapper, pero no para las funciones de Nuxt. Entonces, para nosotros darle este contexto, utilizaremos el metodo de `apply` para darle el contexto que necesita para poder funcionar correctamente: 

```javascript
await wrapper.vm.$options.fetch.apply(wrapper.vm)
```

Con esto, nuestro test ya puede probar hooks de Nuxt, sin embargo, hay que tomar en cuenta que esto solo es util para ver si lo que se realiza dentro del hook no causa errores, no podemos obtener datos directos de ese fetch. vue-test-utils aun no tiene soporte para Nuxt, asi que para unit testing, este metodo funciona. Nuestro test final, se ve de la siguiente forma:

```javascript
it('fetch method works correctly', async () => {
    wrapper = mount(FormFill, {
      localVue,
      vuetify,
      store,
      router,

      mocks: {
        $t: (msg) => msg,
        $fire,
      },
      stubs: { 'client-only': true },
    })

    const spyFetch = jest.spyOn(wrapper.vm.$options, 'fetch')

    // Llamamos al método de submit
    await wrapper.vm.$options.fetch.apply(wrapper.vm)

    expect(spyFetch).toHaveBeenCalledTimes(1)
    expect($fire.firestore.collection).toHaveBeenCalledTimes(1)
  })
```

### Referencias
- [how to spy on dispatch on fetch/middleware method](https://stackoverflow.com/questions/62441267/how-to-spy-on-dispatch-on-fetch-middleware-method)

- [How to test nuxt with jest?](https://stackoverflow.com/questions/65165605/how-to-test-nuxt-with-jest)

- [How to test nuxt.js asyncData and fetch hooks](https://stackoverflow.com/questions/63638214/how-to-test-nuxt-js-asyncdata-and-fetch-hooks)
