
# Cambiar icono de la app

- Primero metemos la imágen dentro de res/drawable-
- Despues en mipmap-v26, click derecho new -> image asset (la imagen debe estar ya creada)
- Ahi buscamos la imagen de nuestro proyecto que queremos
- Seguidamente se refactorizarán el resto de mipmap y ya funciona.
- --------------------------------------------------------------
- Crear la imagen new Vector asset en drawable
- Drawable click derecho new image asset, ahi cambiamos la ruta a la del nuevo icon
  
# Estrucutacion basica de la app
com.pmdm.myapp  
│  
├── models 
│   └── usuario.kt  
├── data 
│   └── mocks  
│   │   ├── RepositoryConverter.kt  
│   │   └── UsuarioRepository.kt 
├── ui  
│   ├── themes  
│   │   └── DarkTheme.kt  
│   ├── views  
│   │   ├── MainActivity.kt  
│   │   └── Fragments.kt  
│  
├── navigation  
│   ├── MainNavGraph.kt  
│   ├── Feature1ScreenRoute.kt  
│   └── Feature2ScreenRoute.kt  
│  
├── composables  
│   ├── Component1.kt  
│   ├── ComposableUiState1.kt (Optional)  
│   ├── Component2.kt  
│   └── ComposableUiState2.kt (Optional)  
│  
├── features  
│   ├── IniciuoEvent
│   ├── InicioScreen
│   ├── InicioUiState
│   ├── InicioViewModel
│   ├── UiStateConverter
│ 
│   ├── feature1  
│   │   ├── Feature1Screen.kt  
│   │   ├── Feature1ViewModel.kt  
│   │   ├── Feature1UiState.kt  
│   │   └── Feature1Events.kt  
│   ├── component1  
│   │   ├── component1.kt  
│   │   ├── component1UiState.kt  
│   │   └── component1Events.kt  
│   └── component2  
│       ├── component2.kt  
│       ├── component2UiState.kt  
│       └── component2Events.kt  

## Mock
- Contendrá la estructura básica de acceso al modelo, igual pero en un data class.
```kt
data class UsuarioMock(
    val id:Int,
    val login: String,

    val password: String
)
```
## DaoMock
- Contiene los accesos a la base de datos ficticia, la estructura es la siguiente:
```kt
class UsuarioDaoMock {
    private var usuarios = inicializaUsuarios()

    private fun inicializaUsuarios():MutableList<UsuarioMock>{
        var usuarios:MutableList<UsuarioMock> = mutableListOf()
        for (i in 0..30) usuarios.add(
            UsuarioMock(
                i,
                generaCadenaAleatoria(7),
                generaCadenaAleatoria(10)
                )
            )
        return  usuarios
    }

    fun get(): MutableList<UsuarioMock> = usuarios
    fun get(login: String): UsuarioMock? = usuarios.find { u -> u.login == login }
    fun insert(usuarioRemoto: UsuarioMock) = usuarios.add(usuarioRemoto)
    fun update(usuarioRemoto: UsuarioMock) {
        val posicion = usuarios.indexOf(get(usuarioRemoto.login))
        if (posicion != -1) usuarios.removeAt(posicion)
        usuarios.add(usuarioRemoto)
    }
    fun delete(id: Int) = usuarios.remove(usuarios.find { it.id==id })
    fun delete(login: String) = usuarios.removeAll { u -> u.login == login }
    private fun generaCadenaAleatoria(tamaño:Int):String{
        var cadena = StringBuilder()
        for (i in 0..tamaño) cadena.append(Random.nextInt(97, 123).toChar())
        return cadena.toString()
    }

}
```
- private var usuarios = inicializaUsuarios() -> inicializa una lista mutable de objetos UsuarioMock que simula una tabla de usuarios en memoria.
- inicializaUsuarios() -> es un metodo que crea los datos ficticios.
- Luego tenemos métodos para el acceso a los datos.

## Repository
- El Repository decide de dónde obtener los datos

```kt
class UsuarioRepository @Inject constructor(private val proveedorUsuarios:UsuarioDaoMock){


    fun get():List<Usuario> = proveedorUsuarios.get().toUsuarios()
    fun get(login:String): Usuario?= proveedorUsuarios.get(login)?.toUsuario()
    fun insert(usuario: Usuario)=proveedorUsuarios.insert(usuario.toUsuarioMock())
    fun update(usuario: Usuario)
    {
        proveedorUsuarios.update(usuario.toUsuarioMock())
    }
    fun delete(id:Int)=proveedorUsuarios.delete(id)
    val numeroUsuarios : Int
        get() = get().size

}
```
- Tiene los mismo métodos que el daomock
- Utilizara el repository converter para transforma los datos daomock a la clase modelo

## ReposritoryConverter
- Transforma datos desde un formato a otro (normalmente de un daomock a un modelo)
```kt
fun UsuarioMock.toUsuario(): Usuario = Usuario(this.id,this.login, this.password)
fun MutableList<UsuarioMock>.toUsuarios(): List<Usuario> =
    this.map { it.toUsuario() }
//endregion

//region Usuario
fun Usuario.toUsuarioMock(): UsuarioMock = UsuarioMock(this.id,this.login, this.password)
```

## UiState

- Data class que se utiliza para representar el estado de la interfaz de usuario
```kt
data class InicioUiState(val id:Int, val login:String, val password:String )
```
- Por lo tanto pongo todos los datos que voy a mostrar de los modelos por pantalla

## UiStateConverter

- Transforma datos "crudos" obtenidos del repositorio en un UiState listo para ser usado por la UI
- Permite transformar un modelo de datos complejo en algo más simple para la pantalla.

```kt
//region Usuario
fun Usuario.toInicioUiState() =
    InicioUiState(this.id,this.login, this.password)

fun List<Usuario>.toUsuarioUiState()=this.map { it.toInicioUiState() }
//endregion

//region InicioUiState
fun InicioUiState.toUsuario() = Usuario(this.id,this.login, this.password)
fun List<InicioUiState>.toUsuarios()=this.map { it.toUsuario() }
//endregion
```

## Event (sealed Interfaz)
- Facilitan el manejo de eventos complejos en el ViewModel.
- El ViewModel recibe un Event y toma una acción específica (como cargar datos, eliminar algo, etc.).
```kt
sealed interface InicioEvent {
    data class Eliminar(val id:Int):InicioEvent
    data class Cambiar(val id:Int):InicioEvent

}
```

## ViewModel
- Procesa los Events que envía el Screen.
- Llama al Repository para obtener datos.
- Actualiza el UiState para que la UI lo observe.

```kt
class InicioViewModel(private val usuarioRepository: UsuarioRepository) : ViewModel() {

    var inicioUiState by mutableStateOf(usuarioRepository.get().toUsuarioUiState())

    fun onInicioEvent(inicioEvent: InicioEvent) {
        when (inicioEvent) {
            is InicioEvent.Eliminar -> {
                usuarioRepository.delete(inicioEvent.id)
                inicioUiState = usuarioRepository.get().toUsuarioUiState()
            }
            is InicioEvent.Cambiar -> {
                // Realizar alguna acción para cambiar el estado
            }
        }
    }
}

```

## Screen 
- El Screen es la función Composable que define cómo se ve la pantalla.
- Recibe siempre un uistate y un event

```kt
@Composable
fun InicioScreen(
    modifier: Modifier = Modifier,
    listaInicioUiState: List<InicioUiState>,
    inicioEvent: (InicioEvent) -> Unit
) {
      LazyColumn (modifier = modifier){
          items(listaInicioUiState)
          {
              LineaLista(inicioUiState = it, inicioEvent = inicioEvent)

          }
      }

 /* LazyColumn(modifier = modifier) {
        itemsIndexed(listaInicioUiState)
        { index, item ->
            LineaLista(posicion = index,inicioUiState = item, inicioEvent = inicioEvent)
        }
    }*/

  /*  LazyColumn(state= rememberLazyListState()) {
        items(items = listaInicioUiState,key={it.id})
        {
            LineaLista(modifier=Modifier.animateItem(),inicioUiState = it, inicioEvent = inicioEvent)
        }
    }*/



}

@Composable
fun LineaLista(
    modifier:Modifier=Modifier,
    posicion: Int = -1,
    inicioUiState: InicioUiState,
    inicioEvent: (InicioEvent) -> Unit
) {
    ElevatedCard(
        modifier = modifier.then(Modifier
            .fillMaxWidth()
            .padding(4.dp)),
        colors = CardDefaults.cardColors(),
        shape = RoundedCornerShape(10),
        elevation = CardDefaults.cardElevation()
    ) {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(15.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            if(posicion!=-1) Text(modifier = Modifier.align(Alignment.Start),text=posicion.toString())
            Text(text = inicioUiState.id.toString())
            Text(text = inicioUiState.login)
       /*     Text(modifier=Modifier.clickable {
                inicioEvent(InicioEvent.Cambiar(inicioUiState.id))
            },text = "*".repeat(inicioUiState.password.length))*/
            Text(text = "*".repeat(inicioUiState.password.length))
            Spacer(modifier = Modifier.fillMaxHeight(0.2f))
            Box(modifier = Modifier.fillMaxWidth()) {
                OutlinedButton(
                    modifier = Modifier.align(Alignment.CenterEnd),
                    onClick = { inicioEvent(InicioEvent.Eliminar(inicioUiState.id)) }) { Text(text = "Eliminar") }

            }
        }
    }
}

@Preview
@Composable
fun InicioScreenPreview() {
    EjemploViewModelTheme {
        val inicioViewModel: InicioViewModel = viewModel()
        Surface {

            InicioScreen(
                listaInicioUiState = inicioViewModel.inicioUiState.toList(),
                inicioEvent = inicioViewModel::onInicioEvent
            )
        }
    }

}
```

# Elevaciones de estado
- Sirven para tener una informacion más centralizada y compartir variables entre diferentes composables
- Ejemplos:
```kt
@Composable
fun ContadorStateful() {
    // Definimos el estado y el manejador del evento click
    var cuentaState by remember { mutableStateOf(0) }
    val onAumentarCuenta: () -> Unit = { cuentaState++ }

    // Pasamos el estado y el manejador al composable ContadorStateless
    ContadorStateless(
        cuentaState = cuentaState,
        onAumentarCuenta = onAumentarCuenta
    )
}
@Composable
fun ContadorStateless(
    cuentaState: Int,
    onAumentarCuenta: () -> Unit) =
    Column {
        // El botón llama al manejar del evento click
        // enviado por el padre
        Button(onClick = onAumentarCuenta) {
            Text("Cuenta Clicks")
        }
        // El texto muestra el estado enviado por el padre
        Text(text = "Llevas $cuentaState Clicks")
    }
```

```kt
@Composable
fun SaludaScreen() {
    // Definimos el estado y los manejadores
    var nombreState by remember { mutableStateOf("") } /*var cuentaState by rememberSaveable { mutableStateOf(0) } si no queremos perder los datos si se recompone la app*/
    val onCambioNombre = { nombre: String -> nombreState = nombre }
    val onClickBorrar = { nombreState = "" }

    Column(horizontalAlignment = Alignment.CenterHorizontally) {
        // Pasamos el estado y manejadores necesarios a los componentes
        IntroduceNombre(
            nombreState = nombreState,
            onCambioNombre = onCambioNombre
        )
        Saluda(
            nombreState = nombreState,
            onClickBorrar = onClickBorrar
        )
    }
}
@OptIn(ExperimentalLayoutApi::class)
@Composable
fun Saluda(
    nombreState: String,
    onClickBorrar: () -> Unit
) {
    FlowRow(
        Modifier.fillMaxWidth().padding(12.dp),
        horizontalArrangement = Arrangement.SpaceBetween
    ) {
        Text(
            modifier = Modifier.padding(12.dp),
            text = "Hola ${nombreState}"
        )
        // Elevamos el evento borrar
        Button(onClick = onClickBorrar) { Text(text = "Borrar") }
    }
}

@Composable
fun IntroduceNombre(
    nombreState: String,
    onCambioNombre: (String) -> Unit
) {
    Row(verticalAlignment = Alignment.CenterVertically) {
        Text(
            modifier = Modifier.padding(12.dp),
            text = "Nombre:"
        )
        TextField(
            value = nombreState,
            // Elevamos el evento cambio en Nombre
            onValueChange = onCambioNombre 
        )
    }
}
```








































