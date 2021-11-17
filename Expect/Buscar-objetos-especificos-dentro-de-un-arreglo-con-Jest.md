Jest tiene varios metodos para encontrar y realizar tests. Podemos buscar dentro de arreglos para ver si tienen algun valor en especifico, o buscar si un objeto tiene alguna propiedad especifica. Pero, como podemos buscar un objeto dentro de un arreglo? Jest no tiene un metodo exacto para hacer esto, sin embargo, podemos encadenar series de metodos para encontrar un objeto dentro de un arreglo y comprobar que este objeto tiene una propiedad especifica. 

En el caso de este proyecto, tenemos la siguiente propiedad dentro de un objeto llamado `configuration`:

```javascript
      configuration: {
        procedures: [
          {
            groups: ['Grupo A'],
            name: 'Procedimiento A',
            tools: [
              { name: 'Firma de consentimiento y asignación a grupo' },
              {
                description: 'Descripcion',
                form: '2',
                name: 'Tool A',
                type: 'Formulario',
              },
            ],
          },

          {
            groups: ['Grupo B'],
            name: 'Procedimiento B',
            tools: [
              { name: 'Firma de consentimiento y asignación a grupo' },
              {
                description: 'Descripcion',
                form: '2',
                name: 'Tool B',
                type: 'Formulario',
              },
            ],
          },
        ],
      },
```

Tenemos el objeto `configuration`, el cual dentro tiene un arreglo llamado `procedures`. Dentro de este arreglo, tenemos dos objetos, que a su vez tiene una propiedad llamada `tools`, el cual es un arreglo de objetos. Lo que queremos comprobar nosotros es que dentro de este arreglo, haya al menos dentro de este arreglo un objeto con la propiedad `type: 'Formulario'`. Primeramente, debemos de encontrar la forma de poder revisar cada uno de los valores dentro del arreglo de `procedures`. Para esto, podemos utilizar la función `beforeEach`, que nos permite realizar una acción para cada uno de los valores dentro del arreglo:

```javascript
    wrapper.vm.procedures.forEach((procedure) =>
      /.../
    )
```

La propiedad `vm` es la instancia de Vue. Con esto podemos acceder a todos los metodos y propiedades de la instancia. Esto nos permite entrar a la propiedad de `procedures` de `configuration` y ciclar entre todos sus valores. 

Ahora, con ayuda de esta propiedad, accedemos a la propiedad de `tools`, en donde nosotros esperamos encontrar un arreglo, que contenga un objeto con la propiedad que queremos. Para probar esto, encadenamos varios "matchers" que buscan esos atributos. Esto en código se mira de la siguiente forma:

```javascript
    wrapper.vm.procedures.forEach((procedure) =>
      expect(procedure.tools).toEqual(
        expect.arrayContaining([
          expect.objectContaining({
            type: 'Formulario',
          }),
        ])
      )
    )
```

Con esto, lo que estamos diciendo es:

1. Esperamos que el arreglo sea igual a
2. Un arreglo que contenga
3. Un objeto que contenga
4. La propiedad `type: 'Formulario'`

Con esto nuestro test ya esta listo para revisar que para todos los `procedures`, todos deben de contener al menos un objeto que tenga la propiedad `type: 'Formulario'`. 

<h3> Como reducir esta consulta </h3>
Como se puede notar, este encadenamiento puede llegar a ser muy confuso o incluso si se tiene que realizar muchas veces, podria contaminar el codigo en general. Para esto, tenemos la opción de `extend`, que nos permite extender los `matchers` que nos proporciona Jest. Por ejemplo, en este caso, podriamos crear un `matcher` que utilice el codigo anterior para que realice esta busqueda dentro de arreglos sin la necesidad de escribir tanto codigo. Esto se veria de la siguiente forma: 

```javascript
expect.extend({
  toContainObject(received, argument) {

    const pass = this.equals(received, 
      expect.arrayContaining([
        expect.objectContaining(argument)
      ])
    )

    if (pass) {
      return {
        message: () => (`expected ${this.utils.printReceived(received)} not to contain object ${this.utils.printExpected(argument)}`),
        pass: true
      }
    } else {
      return {
        message: () => (`expected ${this.utils.printReceived(received)} to contain object ${this.utils.printExpected(argument)}`),
        pass: false
      }
    }
  }
})
```

Con esto, ahora tenemos un matcher que realiza la accion anterior en una sola linea, de forma que  entonces nuestro codigo anterior se reduciria de la siguiente forma:

```javascript
    wrapper.vm.procedures.forEach((procedure) =>
      expect(procedure.tools).toContainObject({'type: 'Formulario','})
```

Como podemos notar, el codigo se hace mucho mas corto. Pero para el test de este proyecto, como solo se utilizo una vez este tipo de test, se decidió utilizar la primera forma para realizar el test.

<h3> Referencias </h3>

- [Jest matching objects in array](https://medium.com/@andrei.pfeiffer/jest-matching-objects-in-array-50fe2f4d6b98)

- [VM - Wrapper Properties](https://vue-test-utils.vuejs.org/api/wrapper/#properties)

- [arrayContaining](https://jestjs.io/docs/expect#expectarraycontainingarray)