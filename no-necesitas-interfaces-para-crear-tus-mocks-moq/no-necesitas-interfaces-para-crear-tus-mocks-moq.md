# No necesitas interfaces para crear tus mocks - Moq

![Moq](https://github.com/moq/moq4) es una de las librerías mas famosas para crear dobles de pruebas para C#.
Esta librería te permite crear _mocks_ de forma muy sencilla utilizando directamente interfaces o clases.

Si vemos los ejemplos que nos expone _Moq_ nos daremos cuenta rápidamente que utiliza interfaces para todos sus _mocks_.

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

Si intentamos crear un _mock_ utilizando directamente una clase, _Moq_ nos lanzará una excepción:

```
Additional information: Can not instantiate proxy of class: [ClassName].
```

Es fácil caer en la tentación de ir por el camino fácil y rápido donde creer que no podemos crear _mocks_ a partir de clases sea nuestra respuesta al problema. Pero si somos un poco curiosos e indagamos en el problemas entenderemos porque esto no es posible.

¿Por qué podemos crear _mocks_ a partir de interfaces?

Cuando creamos un _mock_ a partir de una interfaz lo que está pasando es que estamos creando una implementación de la interfaz. De está forma la librería puede implementar los métodos como más le convenga para darnos la funcionalidad que necesitamos. Ejemplo:

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

Entonces, si esto es lo que pasa cuando creamos un _mock_ a partir de una interfaz.
¿Como se crearán los _mocks_ a partir de clases?

Probablemente ya se te ha venido la respuesta a la cabeza. Sí, se utiliza la herencia para sobreescribir los métodos de la clase y poder cambiar su comportamiento. 

**Aquí está el problema. Por defecto, en C#, todos lo métodos de nuestras clases están protegidos contra sobreescritura (non-virual) por lo que NO podemos cambiar su implementación. Es por esto, por lo que la librería nos lanza una excepción al intentar crear un _mock_ partiendo de una clase con los métodos no virtules.**

Para solucionar el problema es tan sencillo como marcar nuestros métodos como _virtual_. Una vez hecho esto, ya se podrá sobreescribir la implementacion del método y por ende ya podremos crear _mocks_ usando directamente nuestras clases.

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

## Convenciones de equipo. ¿Interfaces para todo?

Es posible caer en el error de estandarizar la práctica de crear una interfaz para todas las clases de las que necesitamos crear dobles de prueba.

Bajo mi punto de vista, esto es un error. 

Las interfaces son un recurso que nos permiten definir un contrato. Obligarnos a seguir una API determinada a la hora de implementer o usar una clase. 
Esto tiene sentido en partes de nuestro de código que son susceptibles a cambiar a lo largo del tiempo o nos interese invertir dependecias, pero no tiene ningún sentido usar interfaces en lugares donde no nos aportan nada más que la posibilidad de crear dobles de pruebas sin tener que marcar los métodos como _virtual_.

Pienso que atacar el factor limitante (en este caso el cómo esta diseñado el lenguaje) y marcar los métodos como _virtual_ es la mejor decisión cuando una interfaz no tiene sentido en nuestro código.

Dejar abierta la posibilidad de sobreescribir un método puede que parezca poco seguro, sin embargo, si priorizamos la composición ante la herencia a la hora de desarrollar no debería ser un problema para nosostros.

En definitiva, pienso que aprovecharse de las interfaces fácilita el trabajo a la hora de crear los _mocks_ pero únicamente debemos usarlas cuando su existencia tiene sentido en el código. Si el código no necesita interfaces, marcar los métodos como _virtual_ es la mejor opción bajo mi punto de vista.

Esta idea podría cambiar si el contexto lo require, por ejemplo, en un proyecto donde la herencia está muy presente es probable que tener los metodos protegidos contra la sobreescritura nos aporte mucho valor.

Cualquier opinión al respecto es bienvenida.
Gracias por leer, un saludo.
 





