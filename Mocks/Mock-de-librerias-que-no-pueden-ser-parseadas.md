Durante uno de los tests de este proyecto, al querer montar el componente a testear, ocurria el siguiente error:

![image](uploads/93c9ed01dc2402ffdcfae263dd404819/image.png)

Este error nos esta diciendo que este tipo de problemas emergen debido a que se esta tratando de importar un archivo que Jest no puede analizar y codificar o "parsear". Es decir, que no es JavasCript puro. Si vemos el componente que estamos tratando de probar, podemos observar que el problema se debe a la siguiente librería que esta siendo importada:

```javascript
<script>
import VFormBase from 'vuetify-form-base'
...
</script>
```

Esta importación no esta dejando que nuestro componente sea montado correctamente, ya que la información contenida en esta, no puede ser analizada por Jest. Como se puede notar, en la misma descripción del error, se nos da a conocer distintas soluciones para poder montar nuestro componente. Después de intentar cada uno de ellos, la que funcionó para nuestro caso, fue el de hacer un mock de esta librería para poder montar nuestro componente. 

Para hacer el mock de esta librería no se utilizo la forma usual de mockear, como se ha visto en casos de [$route](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Mocks/Mock-de-$route-y-$router), [$i18n](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Mocks/Mock-de-$i18n) y [$fire](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Mocks/Mock-de-$fire-(Mock-de-m%C3%A9todos-encadenados)). 

En este caso utilizaremos la función de jest.mock, la cual nos permite hacer un mock de nuestra libreria o API de terceros. Como primer argumento toma el nombre del modulo o libreria que se va a mockear, y como segundo argumento toma la funcion que realizará esta libreria a la hora de ser utilizada o llamada. En nuestro caso, para el mock de `VFormBase`, esto se veria de la siguiente forma: 

```javascript
jest.mock('vuetify-form-base', () => jest.fn())
```

Con esto ya no es necesario colocar nuestro mock en la categoria de mocks dentro de nuestro test, ya que esto se hace globalmente para todos, claro, si este mock se pone de manera global en nuestro archivo. De esta forma, nuestro componente puede ser montado y testeado sin problemas. 

<h3> Referencias </h3>

- [Mock Functions](https://jestjs.io/docs/mock-functions)

- [React Native - Jest.mock must be an inline function. Trouble testing component that uses 'withNavigation'](https://stackoverflow.com/questions/53146079/react-native-jest-mock-must-be-an-inline-function-trouble-testing-component-t)

- [Stubs and Shallow Mount](https://next.vue-test-utils.vuejs.org/guide/advanced/stubs-shallow-mount.html)