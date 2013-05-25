# api-doc

## API REST + Hypermedia

El presente diseño está pensado para ser utilizado como una implementación de una API [REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) haciendo hincapié en la importancia del uso de *Hypermedia*.


## Formato de los documentos

Todos los documentos aquí ejemplificados utilizan el formato *JSON* por motivos de legibilidad y poder expresivo. Si bien es también una recomendación el uso de tal formato para las implementaciones que surjan del presente ejemplo, es relativamente simple llevar estos ejemplos a otros formatos, ya sea *XML*, *HTML* o cualquier otro formato estructurado.


## Hypermedia y atributos generales

Todos los *documentos* (o *recursos*) deben utilizar vínculos de hypermedia para referenciar relaciones externas con otros recursos, o agregaciones con recursos que no necesariamente pertenezcan a los atributos propios del documento, pero que por sentido semántico o expresivo se deban relacionar. Adicionalmente, *todos* los documentos deberán incluir un vínculo a sí mismos bajo la clave `self`.

Todos los vínculos externos deben definirse bajo el atributo `links` de los documentos, utilizando un valor que identifique el tipo de relación que representan (la *clave* de relación) y la referencia al documento externo bajo el atributo `href` de ese vínculo. En general, los vínculos de un documento se estructurarán como se define a continuación:

```json
{
    "links": {
        "relationKey": {
            "href": "/url/to/linked/resource"
        }
    }
}
```

Al ser este un ejemplo acotado de la definición de un vínculo, cualquier otro atributo podría ser agregado según se considere necesario en cada caso, *siempre y cuando esta estructura básica sea mantenida*.

La siguientes claves de relación son *reservadas*:

- `self`: Denota el vínculo que se puede utilizar para obtener nuevamente el mismo recurso. Como se indicó anteriormente, estará presente en *todos* los documentos, y representará la misma dirección mediante la cual el recurso actual fue solicitado, incluyendo cualquier parámetro que pudiera contener.
- `previous`: Utilizado en documentos colectivos con paginación, indica el vínculo a seguir para obtener el documento correspondiente a la *página* anterior de la colección. Estará presente cuando se disponga de una página anterior en una colección paginada.
- `next`: Análogamente a `previous`, es el vínculo que permite obtener el documento correspondiente a la siguiente página de la colección, dadas las opciones de paginación provistas al obtener un documento. Estará presente cuando se disponga de una página siguiente en una colección paginada.
- `full`: Indica el vínculo que se puede utilizar para obtener la versión *completa* de un documento en casos que se haya realizado una solicitud *parcial*. Para mayores detalles, referirse a la sección dedicada a las *respuestas parciales*.


## Colecciones

Toda colección (o *grupo de elementos*) debe utilizar una estructura como se define a continuación:

```json
{
    "entries": [
        {
            "links": {
                "self": {
                    "href": "/some/item.json"
                }
            }
        },
        {
            "links": {
                "self": {
                    "href": "/some/other-item.json"
                }
            }
        }
    ],

    "offset": 0,
    "limit": 10,
    "total": 2,

    "links": {
        "self": {
            "href": "/link/to/self.json"
        }
    }
}
```

En el ejemplo anterior se distinguen los siguientes atributos importantes:

- `entries`: se trata de un `array` de elementos, donde cada uno contiene un vínculo al documento del elemento. Es importante denotar aquí que el vínculo se corresponde con la *representación completa* del documento relacionado, sin modificadores.
- `offset`: valor `integer` que indica el desplazamiento (expresado en cantidad de elementos, comenzando desde `0`) dentro de la colección de elementos, a partir del cual se incluirán `limit` elementos entre los `entries`.
- `limit`: valor `integer` que indica la cantidad máxima de elementos a incluir entre los `entries`.
- `total`: valor `integer` que indica la cantidad total de elementos disponibles en la colección.

Dado que las colecciones representan tipos de recursos o agrupaciones lógicas de los mismos, las mismas deben utilizar URIs sencillas siguiendo el formato:

    /<resource-type-in-plural>.json

Por ejemplo, para referenciar el conjunto de datos de personas (`people`, en inglés) se utilizará la siguiente URI:

    /people.json

La *paginación* antes mencionada se logra utilizando parámetros mediante el *query string* de la URI de un recurso. Específicamente, los parámetros utilizables son `offset` y `limit`. El primero indicará el desplazamiento dentro de la colección de elementos ("el índice de inicio de la página"), mientras que el segundo indicará la cantidad máxima de elementos (`entries`) a incluir en un recurso ("el tamaño de la página"). Las siguientes URIs ejemplifican casos de paginación:

    /people.json?offset=15
    /people.json?offset=20&limit=20

En los casos que un documento colectivo deba utilizar paginación, incluirá hasta 2 vínculos adicionales que permitirán recorrer la lista completa de elementos mediante el uso de hypermedia: `previous` y/o `next`.

- Si el `offset` actual es mayor que `0` y se dispone de más de 1 elemento, la *página* actual no será la primera de la colección, por lo que se incluirá un vínculo `previous` (que debe incluir cualquier [*modificador de recurso*](#modificadores-de-recurso)) con el `offset` de la página anterior (teniendo en cuenta el `limit` especificado) denotando que la página no es la primera en la colección.
- Si el `offset` actual es tal que la página actual no es la última en la colección, se incluirá un vínculo `next` (que debe incluir cualquier [*modificador de recurso*](#modificadores-de-recurso)) con el `offset` de la página anterior (teniendo en cuenta el `limit` especificado) que marcaráque la página no es la última de la colección.

A continuación se presenta un ejemplo que muestra una combinación de las situaciones de paginación antes descriptas:

```json
{
    "entries": [
        ...
    ],

    "offset": 15,
    "limit": 15,
    "total": 33,

    "links": {
        "self": {
            "href": "/people.json?offset=15&limit=15"
        },
        "previous": {
            "href": "/people.json?offset=0&limit=15"
        },
        "next": {
            "href": "/people.json?offset=30&limit=15"
        }
    }
}
```


## Documentos individuales


## Modificadores de recurso


## Tipos de datos

Los valores de los atributos deben representarse *consistentemente*. A continuación se brinda una lista con las representaciones estándar para los tipos de datos más comunes:

### Números

Los números pueden ser expresados como valores `integer` (`0`, `200`, `4019`), como valores `float` (`0.5`, `3.0`, `68622.42`), o como valores de precisión y longitud arbitrarias mediante el empleo de `string` (`"8126318281987921632167"`, `"921763.8126326178321531245632571"`).

### Identificadores

Los identificadores de clave primaria deben estar representados como `string`, aún siendo números pequeños enteros. Esto asegurará la consistencia entre distintos tipos de recursos.

Ejemplos válidos de identificadores son: `"1"`, `"105"`, `"18291278321678421678321678321"`, `"my-id"`, or "abc-123-zyx-987"`.

### Booleanos

### String

### Fecha

### Hora

### Timestamp (fecha y hora)

### URIs/URLs

Las referencias a recursos deben ser *relativas*. Esto quiere decir que no se debe incluir información del host donde se ubiquen los recursos en sus URIs.

La siguiente es una URI **válida**:

    /api/books/5678123.json

Mientras que esta **no lo es**:

    http://server.example.com/api/books/5678123.json


## Extensiones funcionales


## Sobre este ejemplo

El presente ejemplo intenta modelar una estructura básica de API que respete los lineamientos definidos en este proyecto.

### Recursos

Los siguientes recursos están disponibles:

- `/articles.json`
- `/editions.json`
- `/sections.json`
- `/supplements.json`
- `/tags.json`

Estos son los recursos colectivos que soportan el verbo HTTP `GET` y retornan una colección paginada de elementos, siguiendo la estructura especificada en [la sección referente a colecciones](#colecciones).
