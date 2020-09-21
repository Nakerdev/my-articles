# No necesitas interfaces para crear tus mocks - Moq

![Moq](https://github.com/moq/moq4) es una de las librerías mas famosas de dobles de pruebas para C#.
Esta librería te permite crear tus _mocks_ de forma muy sencilla utilizando directamente interfaces o clases.

Si vemos los ejemplos que nos expone _Moq_ nos daremos cuenta rápidamente que utiliza interfaces para todos
sus _mocks_.

```
var mock = new Mock<ILoveThisLibrary>();

mock
    .Setup(library => library.DownloadExists("2.0.0.0"))
    .Returns(true);

public interface ILoveThisLibrary
{
    bool DownloadExists(string version);
}
```

Si intentamos crear un _mock_ utilizando directamente una clase veremos que _Moq_ nos lanza una excepción.

```
Additional information: Can not instantiate proxy of class: ClassName.
```

Es fácil caer en la tentación de ir por el camino fácil y rápido donde creer que no podemos 
crear _mocks_ de clases sea nuestra respuesta al problema. Pero si somos un poco curiosos e indagamos en el
problemas entenderemos porque esto no es posible.

¿Por qué podemos crear _mocks_ a partir de interfaces?

Cuando creamos un _mock_ a partir de una interfaz lo que está pasando es que estamos creando una implementación 
de la interfaz. De está forma la librería puede implementar los métodos como más le convenga para darnos la funcionalidad
que necesitamos. Ejemplo:

Crear un _mock_ con _Moq_:

```
var mock = new Mock<ILoveThisLibrary>();

mock
    .Setup(library => library.DownloadExists("2.0.0.0"))
    .Returns(true);
```

Es lo mismo que hacer lo siguiente:

```
var mock = new LoveThisLibraryMock();

public class LoveThisLibraryMock : ILoveThisLibrary 
{
    public bool DownloadExists(string version)
    {
        return true;
    }

}
```

Entonces, sí esto es lo que pasa cuando creamos un _mock_ a partir de una interfaz.
¿Como se crearán los _mocks_ a partir de clases?

Probablemente ya se te ha venido la respuesta a la cabeza. Sí, se utiliza la herencia para sobreescribir los 
métodos de la clase y poder cambiar su comportamiento. 

**Aquí está el problema. Por defecto, en C#, todos lo métodos de nuestras clases son no virtuales
por lo que NO podemos cambiar su implementación. Es por esto, por lo que la librería nos lanza una excepción al intentar crear
un _mock_ partiendo de una clase con los métodos no virtules.**

Para solucionar el problema es tan sencillo como marcar nuestros métodos como _virtual_. Una vez hecho esto, ya se podrá sobreescribir
la implementacion del método y por ende ya podremos crear _mocks_ usando directamente nuestras clases.

## Clases con dependencias

Si a la clase que estamos usando para crear el _mock_ se le inyectan dependencias por el constructor, _Moq_ se nos quejará.
Necesitaremos pasarle las dependencias de la clase a la hora de instanciar el _mock_. Estas dependecias no son importantes ya
que no se usarán en la implementación, pero son necesarias si no tenemos un constructor sin parámetros.
Para instanciar el _mock_ podemos pasar _null_ a todas sus dependecias o utilizar la utilidad de _Moq_ _It.IsAny_.

```
var mock = new Mock<ILoveThisLibrary>(
    It.IsAny<IDependecy1>,
    It.IsAny<Dependecy2>)
```

o

```
var mock = new Mock<ILoveThisLibrary>(null, null)
```

Personalmente me gusta más la primera opción.




