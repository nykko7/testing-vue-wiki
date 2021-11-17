En entradas anteriores hemos hablado de como utilizar espías y como estos nos permiten ver si una función es llamada correctamente y en ciertos casos, revisar si esta función fue llamada con algun valor exacto. 

Ahora bien, que sucede cuando tenemos llamadas a un método que hace llamada a una función asincrona y queremos revisar si esta función fue llamada correctamente? Si tratamos de hacerlo de la forma que ya conocemos, puede que nuestro test no funcione debido a que la llamada o la promesa no ha sido resuelta. Tomemos por ejemplo el siguiente método:

```javascript
import { changelogService } from '~/services/index.js'
    async addProject() {
      try {
        ...
        const body = {
          date: new Date(),
          user: this.$fire.auth.currentUser.email,
          type: 'newProject',
          prev: '',
          newChange: this.name,
        }
        await changelogService(this.$fire, body, id)
        ...
    },
```

Al hacer clic en el botón que hace al llamado de este metodo, este tiene dentro otra función llamada changelogService, que es la que deseamos espiar. Si hacemos esto normalmente: 

```javascript
    const spyChangelog = jest.spyOn(services, 'changelogService')

    const addButton = wrapper.find('#createProject')
    addButton.trigger('click')
    await wrapper.vm.$nextTick()

    expect(spyChangelog).toHaveBeenCalledTimes(1)
```

Nos mostrara que nuestro test falla debido a que no se llamo ninguna vez el método. Sin embargo, nuestro componente funciona y realiza el llamado correctamente al probarlo localmente o en produccion. Esto puede suceder debido a que si hay una promesa anterior la cual no ha sido resuelta, nuestro test no alcanza a ver el disparo de la llamada que estamos espiando. En nuestro caso, era una promesa de firebase. Para resolver promesas que se quedan esperando en nuestros tests, podemos definir la siguiente constante: 

```javascript
const flushPromises = () => new Promise(setImmediate)
```

Lo cual nos resuelve cualquier promesa en nuestro test inmediatamente. En nuestro caso, como solamente estamos espiando que la llamada se haga correctamente, no hay problema con que nuestras promesas se hagan inmediatamente. Antes de hacer nuestros tests con `expect`, llamamos a nuestra constante/funcion:

```javascript
await flushPromises()
```

Con esto, las promesas seran resueltas antes de nuestro test y asi podremos observar cualquier llamada de nuestro espia sin problema. Es importante mencionar que esta constante se debe de declarar en test individuales y solo donde se necesite, ya que esta podria afectar a otros tests. Nuestro test queda entonces de la siguiente forma:

```javascript

  it('Creating a project triggers changelog service correctly', async () => {
    const flushPromises = () => new Promise(setImmediate)
    wrapper = mount(newC, {
      localVue,
      vuetify,
      store,

      mocks: {
        $t: (msg) => msg,
        $fire,
      },

      data() {
        return {
          name: 'Proyecto B',
        }
      },

      attachTo: document.body,
    })

    const spyChangelog = jest.spyOn(services, 'changelogService')

    const addButton = wrapper.find('#createProject')
    addButton.trigger('click')
    await wrapper.vm.$nextTick()

    await flushPromises()

    expect(spyChangelog).toHaveBeenCalledTimes(1)
  })
```

### Referencias
- [How to make Jest wait for all asynchronous code to finish execution before expecting an assertion](https://stackoverflow.com/questions/44741102/how-to-make-jest-wait-for-all-asynchronous-code-to-finish-execution-before-expec)