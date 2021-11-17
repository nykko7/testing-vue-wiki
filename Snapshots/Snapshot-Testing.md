Como podemos comprobar que cierta informacion que queremos que se muestre en nuestra pantalla sea la correcta? Una forma de hacerlo es buscar en nuestro DOM cada lugar donde este la informacion que mostramos y verificar que esta coincida con la informacion que nosotros le dimos a nuestra app. Sin embargo, eso puede ser tedioso y consumir mucho tiempo si se quieren hacer varias comprobaciones. Para esto, Jest nos da una herramienta muy poderosa llamada snapshots.

Un test de snapshot típico renderiza un componente, toma un snapshot y luego la compara con un archivo de snapshots de referencia almacenado en el proyecto. El test fallará si las dos snapshots no coinciden, esto puede ser porque el cambio es inesperado o el snapshot de referencia debe actualizarse a la nueva versión del componente.

Para realizar un snapshot, utilizamos el metodo `toMatchSnapshot()`. Este metodo nos permite generar un .snapshot donde se almacena el codigo que fue renderizado al montar nuestro componente. Ahora bien, como comprobamos que nuestro componente es igual a este snapshot? `Wrapper` tiene un metodo llamado `html()`, este metodo nos devuelve el html de nuestro wrapper, de forma que este deberia de coincidir con el snapshot tomado con el metodo `toMatchSnapshot()`. El test entonces se ve de la siguiente forma:

```javascript
   expect(wrapper.html()).toMatchSnapshot()
```

Si revisamos nuestro snapshot, podemos observar si esta presente la informacion que plasmamos en el DOM. Por ejemplo, en la configuracion del proyecto, se coloco como nombre de la enfermedad "Enfermedad X", entonces esto deberia de estar presente en nuestro snapshot:

```html
       <div class=\\"row\\">
           <div class=\\"d-flex align-center col\\">
              <p>
                 Enfermedad X
              </p>
           </div>
       </div>
```

Con esto podemos observar que nuestra informacion fue correctamente impresa en el DOM. Ahora bien, si en algun momento la UI cambia, el test va a fallar. Sin embargo, los datos deberian de imprimirse de igual forma. Entonces mientras eso se siga cumpliendo, para actualizar el snapshot al correr los tests en nuestra terminal, corremos el siguiente comando:

```shell
yarn jest testName.spec.js -u
```

Con el comando -u, actualizamos nuestro snapshot y de nuevo el test pasa.

<h3> Referencias </h3>

- [Snapshot Testing](https://jestjs.io/docs/snapshot-testing)