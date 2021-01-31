# Rocket Overview

## Routing

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