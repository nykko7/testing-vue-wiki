Durante uno de los tests de este proyecto, se testeó el siguiente método:

```javascript
    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
```

Este método utiliza la función `Object.assign`, lo que hace es copiar las propiedades del segundo objeto (dado como argumento) al primero. En este caso, se copiaran las propiedades de `this.defaultItem` al objeto `{}`. Ahora bien, en el caso de nuestro test, `this.defaultItem` no tiene propiedades, entonces en nuestro test, nosotros esperamos obtener un arreglo vacío. Nuestro expect se veria de la siguiente forma:

```javascript
expect(wrapper.vm.editedItem).toBe({})
```

Sin embargo, esto nos presenta con el siguiente problema:

```shell
“Received: serializes to the same string”
```

Esto significa que lo que se recibió coincide si ambos fueran un string. Sin embargo, si le damos a nuestro expect `"{}"`, el test seguirá fallando, porque recibe un objeto, no un string de un objeto. Que podemos hacer entonces?

Como sabemos, Jest tiene otro tipo de métodos en `expect`, uno de ellos siendo `toStrictEqual`. Este método compara todas las propiedades de instancias de objetos, y es conocido como "deep equiality". El método de `toBe` no nos funciona en este caso dado que los objetos son diferentes instancias. `toBe` es útil cuando comparamos strings, numeros o booleanos, pero para tests como funciones u objetos, es mejor utilizar `toStrictEqual`. Si hacemos ese cambio nuestro test pasa sin problemas.

El test completo es el siguiente:

```javascript
  it('closeDelete method works correctly', async () => {
    wrapper = mount(index, {
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
          forms: [
            { id: '1', pregunta: 'Respuesta 1' },
            { id: '2', pregunta: 'Respuesta 2' },
          ],
        }
      },
    })

    const delCanAnswSpy = jest.spyOn(wrapper.vm, 'closeDelete')

    const delCanAnswBtn = wrapper.find('#cancelBtn')
    delCanAnswBtn.trigger('click')
    await wrapper.vm.$nextTick()

    expect(delCanAnswSpy).toHaveBeenCalledTimes(1)
    expect(wrapper.vm.dialogDelete).toBe(false)
    expect(wrapper.vm.editedIndex).toBe(-1)
    expect(wrapper.vm.editedItem).toStrictEqual({})
  })
```

### Referencias
- [Jest.js error: “Received: serializes to the same string”](https://stackoverflow.com/questions/56839801/jest-js-error-received-serializes-to-the-same-string)
- [What is the difference between 'toBe' and 'toEqual' in Jest?](https://stackoverflow.com/questions/45195025/what-is-the-difference-between-tobe-and-toequal-in-jest)
- [toEqual](https://jestjs.io/docs/expect#toequalvalue)
- [Object.assing](https://www.educative.io/edpresso/what-is-the-objectassign-method-in-javascript?affiliate_id=5082902844932096&utm_source=google&utm_medium=cpc&utm_campaign=grokking-ci&utm_term=&utm_campaign=Grokking+Coding+Interview+-+USA%2B&utm_source=adwords&utm_medium=ppc&hsa_acc=5451446008&hsa_cam=1871092258&hsa_grp=84009716779&hsa_ad=396821895536&hsa_src=g&hsa_tgt=aud-597782228586:dsa-1287243227899&hsa_kw=&hsa_mt=b&hsa_net=adwords&hsa_ver=3&gclid=CjwKCAjwz_WGBhA1EiwAUAxIcWjk7oqKxOZLn-Jk160m0wEbX3er2NNylAKU4QTDGn1ltzuqgoeToRoCJ6cQAvD_BwE)