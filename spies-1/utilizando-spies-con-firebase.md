En entradas anteriores, ya se ha hablado de como se puede hacer un mock para el modulo de `firebase`. En los casos en los que se habia utilizado, era para que solamente los tests funcionaran y el componente pudiera ser montado correctamente. Ahora lo que queremos nosotros es probar que estas funciones son llamadas correctamente. 

Firebase no tiene forma de testear sus funciones localmente, entonces, como podemos nosotros testar nuestros componentes que utilizan esta infomación? Una forma de hacerlo es utilizando los espias o `spies` de Jest.

### Que es un espia?

Un espía crea una función mock similar a jest.fn pero también esta atento al numero de veces que es llamado el método de un objeto o componente. Estos son útiles para cuando se tienen funcionalidades de API de terceros, como lo es el caso de firebase. 

Para poder utilizarlo, aun es necesario hacer el mock de esta libreria, sin embargo, ahora esta será testeada para ver si esta es llamada correctamente y el numero de veces que debe de ser llamada dependiendo de como es llamada.

### Espias en acción

Veamos un ejemplo. A continuación tenemos el método de logIn, que permite acceder al usuario a nuestra aplicación:

```javascript
  methods: {
    async login() {
      try {
        this.loading = true
        await logIn(this.$fire, {
          email: this.email,
          password: this.password,
        })
      } catch (error) {
        this.loading = false
        this.$store.dispatch('notifications/open', {
          color: 'error',
          text: error.message,
        })
      }
    },
  },
```

Como podemos notar, se tiene el método de `logIn` de las funciones de firebase, el cual se ve de la siguiente forma:

```javascript
export const logIn = ($fire, { email, password }) => {
  return $fire.auth.signInWithEmailAndPassword(email, password)
}
```

Antes de utilizar nuestro espia, primero tenemos que hacer el mock de esta función:

```javascript
const $fire = {
  auth: {
    signInWithEmailAndPassword: jest.fn(),
  },
}
```

> Para ver como se realiza el mock de este tipo de funciones, entra [aqui](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Mocks/Mock-de-$fire-(Mock-de-m%C3%A9todos-encadenados)). 

Una vez que ya hemos realizado el mock de nuestra libreria, ahora podemos usar esta libreria para revisar que esta es llamada correctamente a la hora de realizar cierta acción. Aqui es donde entra el metodo de `spyOn`. Este metodo acepta dos argumentos, uno siendo la instancia que queremos espiar, y el metodo el cual queremos revisar. En nuestro caso, se veria de la siguiente forma:

```javascript
    const spylogIn = jest.spyOn(wrapper.vm, 'login')
```

Ahora, para revisar si este metodo fue llamado de forma correcta, primero tenemos que interactuar con la accion que dispara esta llamada. En nuestro caso, se hace mediante el click de un botón:

```javascript
    const logInButton = wrapper.find('#logBtn')
    logInButton.trigger('click')
    await wrapper.vm.$nextTick()
```

Al hacer click, nuestro método debió de ser llamado, lo cual podemos comprobar con el método `toHaveBeenCalledTimes` de Jest:

```javascript
    expect(spylogIn).toHaveBeenCalledTimes(1)
```

Con esto comprobamos que nuestro método es llamado cuando debe y que este funciona con nuestro mock de firebase correctamente. 

### Referencias
- [jest.spyOn](https://jestjs.io/docs/jest-object#jestspyonobject-methodname)
- [How to mock the axios library in jest without mock adapter or library](https://techdoma.in/react-js-testing/how-to-mock-the-axios-library-in-jest-without-mock-adapter-or-library)