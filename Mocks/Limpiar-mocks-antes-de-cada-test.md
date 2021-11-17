En algunos de los tests de este proyecto, se revisa varias veces si cierto método es llamado al hacer clic en varios y distintos botones. Esto causaba un problema, ya que el primer test si pasaba (el que esperaba que fuera llamado una vez), pero los que le seguian, tenian registrado que el método fue llamado mas de una vez:

![image](uploads/5354b2a006e17fa40794096f0e5b865f/image.png)

Porque sucede esto a pesar de que nuestros test tienen wrappers independientes para cada uno de ellos? Esto sucede debido a que el espia tenia el mismo nombre en todos nuestros tests:

```javascript
const spyChangelog = jest.spyOn(services, 'changelogService')
```

Entonces, al realizar los tests anteriores, jest guardaba el numero de veces que este método había sido llamado, sin importar que cada uno de nuestros espías era re-definido en nuestros tests. Para arreglar esto, usaremos la función de `afterEach` que nos permite hacer acciones después de cada uno de nuestros tests. En este caso, borraremos el contenido de llamdas de nuestro espía, esto lo hacemos mediante el método `jest.clearAllMocks()`. Esto nos limpia nuestros mocks y espías y nos regresa el numero de llamadas a 0. Nuestro test entonces se vería de la siguiente forma:

```javascript 
describe('index page tests', () => {
   
  ...

  afterEach(() => {
    jest.clearAllMocks()
  })

  ...

  it('Deleting a colaborator triggers changelog service correctly', async () => {
    const wrapper = mount(indexC, {
      localVue,
      vuetify,
      store,

      mocks: {
        $t: (msg) => msg,
        $route,
        $fire,
      },
      stubs: ['router-link'],

      data() {
        return {
          users: [
            {
              email: 'joker@mail.com',
              id: 'joker@mail.com',
              role: 'superAdmin',
            },
            { email: 'mona@mail.com', id: 'mona@mail.com', role: 'admin' },
          ],
        }
      },
    })

    wrapper.vm.deleteItem(0)

    const spyChangelog = jest.spyOn(services, 'changelogService')

    const delConfButton = wrapper.find('#confirmDel')
    delConfButton.trigger('click')
    await wrapper.vm.$nextTick()

    expect(spyChangelog).toHaveBeenCalledTimes(1)
  }) 

```

Con esto, solucionamos el problema de llamadas de nuestros espías. 

### Referencias 
- [How to reset Jest mock functions calls count before every test](https://stackoverflow.com/questions/47812801/how-to-reset-jest-mock-functions-calls-count-before-every-test)