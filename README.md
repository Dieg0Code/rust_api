# Rocket Overview

Rocket hace un uso abundante de las extensiones de sintaxis de Rust y otras funciones avanzadas e inestables. Debido a esto, necesitaremos usar la **nightly** 

## Routing

##### fuente original [aqui](https://rocket.rs/v0.4/overview/)

La tarea principal de Rocket es enrutar las solicitudes entrantes al handler/manejador apropiado usando las rutas declaradas en la aplicación. Las rutas son declaradas usando el "Rocket's route attribute". El atributo describe la solicitud que coincide con la ruta. El atributo es colocado sobre una función la cual es el "request handler/manejador de solicitud" para esa ruta.

Ejemplo de una ruta simple

```Rust
#[get("/")]
fn index() -> &'static str {
   "Hello, world!"
}
```

Esta  ruta, llamada `index`, hará coincidir la solicitud HTTP `GET` entrante con el path `/`, el index. El "request handler/manejador de solicitudes" retorna una string. Rocket usara la string como el body de una repuesta HTTP completamente formada.

## Dynamic Params

Rocket te permite interpretar segmentos de una ruta de solicitud de forma dinámica. Para ilustrar, vamos a usar la siguiente ruta:

```Rust
#[get("/hello/<name>/<age>")]
fn hello(name: String, age: u8) -> String {
    format!("Hello, {} year old named {}!", age, name)

```

La ruta `hello` anterior coincide con dos "dynamic path segments" declarados dentro de brackets en el path: `<name>` y `<age>`. "Dynamic" significa que el segmento puede ser cualquier valor que el usuario final desee.

Cada parámetros dinámicos (`name` y `age`) debe tener un tipo, en este ejemplo `&str` y `u8`, respectivamente. Rocket intentará parsear la string (to parse the string), Rocket usa el FromParam trait, el cual puedes implementar para tus propios tipos.

## Handling Data

La data del body de una solicitud es manejada de forma especial en Rocket: via el FromData trait. Cualquier tipo que implemente `FromData` puede ser derivado de la data del body entrante. Para decirle a Rocket que estas esperando una body data request, el argumento `data` route es usado con el nombre del parámetro en el "request handler/manejador de solicitud"

```Rust
#[post("/login", data = "<user_form>")]
fn login(user_form: Form<UserLogin>) -> String {
   format!("Hello, {}!", user_form.name)
}
```

La ruta `login` de arriba dice que esta esperando `data` de tipo `From<UserLogin>` en el parámetro `user_form`. El tipo **Form** es un tipo integrado de Rocket que sabe cómo parsear (knows how to parse) forms en estructuras. Rocket intentara automáticamente parsear el request body en el `form` y llamar al `login` handler si es que el parsing tiene éxito. Otros tipos `FromData` incorporados son `Data`, `Json`, y `Flash`.

## Request Guards

Adicional a dynamic path y data parameters,los request handlers pueden también contener un tercer tipo de parámetro: requests guards. Request guards no son declarados en el route attribute, y cualquier numero de ellos puede aparecer en el request handler signature.

Los Request guards protegen al handler de la ejecución a menos que los metadatos de la solicitud entrante cumplan un conjunto de condiciones. Por ejemplo, si está escribiendo una API que requiere que las llamadas sensibles vayan acompañadas de una clave API en el encabezado de la solicitud, Rocket puede proteger estas llamadas por medio de una custom `ApiKey` request guard:

```Rust
#[get("/sensitive")]
fn sensitive(key: ApiKey) { ... }
```

`ApiKey` protege el `sensitive` handler de ejecutarse incorrectamente. ara que Rocket llame al `sensitive` handler, el `ApiKey` type necesita ser derivado mediante una implementación `FromRequest`, la cual en este caso, valida el API header, Request guard es un poderoso y único concepto de Rocket.

## Responders

El tipo que retorna un request handler puede ser cualquier tipo que implemente Responder:

```Rust
#[get("/")]
fn route() -> T { ... }
```

Arriba, T debe implementar `Responder`. Rocket implementa `Responder` para varios de los tipos de la biblioteca estándar incluyendo `&str`, `String`, `File`, `Option`, y `Result`. Rocket también implementa responders personalizados como **Redirect**, **Flash** y **Template**.

La tarea de un `Responder` es generar un `Response`, si es posible. Los `Responder` pueden fallar con un status code. Cuando lo hacen, Rocket llama el correspondiente error catcher, un `catch` route, el cual puede ser declarado de la siguiente forma:

```Rust
#[catch(404)]
fn not_found() -> T { ... }
```

## Launching

Para que Rocket comience a enviar solicitudes a las rutas, las rutas necesitan ser montadas. Después de ser montadas, la aplicación debe ser lanzada, estos dos pasos usualmente se hace en el `main`:

```Rust
rocket::ignite()
   .mount("/base", routes![index, another])
   .launch();
```

El `mount` call toma una ruta base y un conjunto de rutas a través la macro `routes!`. La ruta base (`/base` de arriba) se antepone a la ruta en la lista. Esto efectivamente asigna nombres a las rutas, lo que permite una composición más sencilla.

La `launch` call inicia el server. En desarrollo, Rocket imprime informacion util en la consola para hacerte saber que todo esta bien.

```bash
🚀  Rocket has launched from http://localhost:8000
```