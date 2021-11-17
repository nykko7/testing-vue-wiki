Anteriormente ya se ha hablado de que son los [espias](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Spies/Utilizando-spies-con-firebase) y como estos funcionan para verificar si un metodo fue llamado correctamente, ademas de tener la opcion de revisar si ha sido llamado mas de una vez. En esta entrada se hablará de un pequeño problema que esta herramienta tiene y como se soluciona.

Lo que queremos probar, es que al dar click en el botón de `cancel`, este llame al metodo `closeDelete` correctamente. En nuestro template, esto se ve de la siguiente forma:

```html
          <v-btn
               id="cancelBtn"
               color="secondary"
               text
               @click="closeDelete"
           >
```

Configuramos nuestro espía de la forma que ya conocemos: 

```javascript
    const delCanFormSpy = jest.spyOn(wrapper.vm, 'closeDelete')

    const delCanFormBtn = wrapper.find('#cancelBtn')
    delCanFormBtn.trigger('click')
    await wrapper.vm.$nextTick()

    expect(delCanFormSpy).toHaveBeenCalledTimes(1)
```
Sin embargo esto nos causa un problema, nuestro `expect` falla, porque es esto? Despues de un poco de investigación y experimentación, se encontró que Jest a veces tiene problemas para encontrar llamadas a funciones/metodos que no tienen los parentesis en sus llamadas. Es decir, en la linea de nuestro template: 

```html
          <v-btn
               ...
               @click="closeDelete()"
           >
```

Es necesario que la llamada a cualquier método o función tenga los parentesís, incluso si este no tiene argumentos. Esto permite que nuestro espia pueda encontrar nuestro método y asi nuestro test pasa. Este es un pequeño problema con los espías que se puede evitar facilmente si hacemos nuestras llamadas con los paréntesis, y que nos puede ahorrar muchos problemas a la hora de testear y debuggear. 

### Referencias 

- [Vue Testing (JEST): button.trigger('click') not working](https://stackoverflow.com/questions/62828904/vue-testing-jest-button-triggerclick-not-working)