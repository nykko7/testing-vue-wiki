En uno de los componentes de este proyecto, se tiene el siguiente código a la hora de montar nuestro componente:

```javascript
mounted{
    try {
      self.loadingForms = true
      await this.$fire.firestore
        .collection(`projects/${this.$route.params.id}/forms`)
        .where('name', '==', 'Registro de participantes')
        .get()
        .then((querySnapshot) => {
          if (querySnapshot.size > 0) {
            self.isParticipantsFormExists = true
          }
          self.loadingForms = false
        })
        .catch((error) => {
          // eslint-disable-next-line no-console
          self.loadingForms = false
          console.error('Error getting documents: ', error)
        })
      if (self.forms && self.forms.length > 0) {
        self.registerFormId = self.forms[0].id
      }
    } catch (error) {
      console.error('Error getting participants form', error)
      self.loadingForms = false
    }
}
```

A la hora de montar nuestro componente en nuestro test, este nos genera un error debido a que a la hora de montarlo, nuestro test no sabe acerca de la libreria `$fire`. Es entonces necesario mockear esta libreria. Sin embargo, esto es distinto a mocks anteriores como los de `$i18n` o `$route`. Esto se debe a que al metodo firestore se le aplican una cadena de metodos, en donde cada uno de ellos necesita de la informacion que retorna cada metodo para funcionar. Entonces para mockearlo, nosotros también tenemos que mockear este encadenamiento de metodos. Para esto, podemos utilizar de nuevo las funciones mock, pero ahora utilizando sus argumentos. Aqui lo que hacemos es que al mockear uno de los metodos, este nos devuelva el metodo que sigue en la cadena, de esta forma al final $fire nos podra retornar algo y todos los métodos estaran mockeados. Un ejemplo corto seria el siguiente. Para la siguiente cadena de metodos:

```javascript
zoomOut(callback) {
        // Zooms out the current screen
        this.view.current.zoomOut(300).done(() => {
            (hasCallback(callback)) && callback();
        });
    }
```

El mock se veria de la siguiente forma:
```javascript
    instance.view = {
        current: {
            zoomOut: jest.fn(() => {
                return {
                    done: jest.fn((callback) => {
                        callback();
                    })
                };
            })
        }
    };
```

En este caso, podemos notar que el método de `done` si nos devuelve información, pero esto no es necesario. Esto se realiza en el caso en el que queremos observar o espiar si nuestro mock fue llamado correctamente.

Ahora bien, para nuestro caso, el mock se veria de la siguiente forma:
```javascript
const $fire = {
  firestore: {
    collection: jest.fn(() => {
      return {
        where: jest.fn(() => {
          return {
            get: jest.fn(() => {
              return {
                then: jest.fn(() => {
                  return {
                    catch: jest.fn(),
                  }
                }),
              }
            }),
          }
        }),
      }
    }),
  },
}
```
Como podemos notar, el método de `then` nos retorna la función mock de `catch`, la cual a su ves es el retorno del método `get`, y asi sucesivamente hasta que llegamos a nuestro problema inicial, que era `$fire`. Con esto podemos mockear `$fire` y en general cualquier tipo de metodos encadenados. 

<h3> Referencias </h3>

- [How do I mock and test chained function with jest?](https://stackoverflow.com/questions/57719741/how-do-i-mock-and-test-chained-function-with-jest)

- [How do I mock this method chain in Jest?](https://stackoverflow.com/questions/51922895/how-do-i-mock-this-method-chain-in-jest)

- [Mocking Chained APIs in Jest](https://blog.bguiz.com/2017/mocking-chained-apis-jest/)