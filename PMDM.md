Table of Contents
=================

* [Cambiar icono de la app](#cambiar-icono-de-la-app)
* [Estrucutacion basica de la app](#estrucutacion-basica-de-la-app)
   * [Mock](#mock)
   * [DaoMock](#daomock)
   * [Repository](#repository)
   * [ReposritoryConverter](#reposritoryconverter)
   * [UiState](#uistate)
   * [UiStateConverter](#uistateconverter)
   * [Event (sealed Interfaz)](#event-sealed-interfaz)
   * [ViewModel](#viewmodel)
   * [Screen](#screen)
* [Elevaciones de estado](#elevaciones-de-estado)
* [Maqueado de UI](#maqueado-de-ui)
   * [Surface](#surface)
   * [Box](#box)
   * [Column y Row](#column-y-row)
   * [FlowColumn y FlowRow](#flowcolumn-y-flowrow)
   * [Imagenes](#imagenes)
* [MD3](#md3)
   * [Ejemplo botón like](#ejemplo-botón-like)
   * [Checkbox](#checkbox)
   * [Cards](#cards)
   * [Textfiel](#textfiel)
   * [Chip](#chip)
   * [Dialogos](#dialogos)


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

# Maqueado de UI
## Surface
- Sirve para darle atributos a elementos que no los pueden tener
```kt
@Composable
fun TextoConForma(
    modifier: Modifier = Modifier,
    texto : String = "Hola Mundo",
    color : Color = MaterialTheme.colorScheme.primary) {
    Surface(
        modifier = modifier.then(Modifier.padding(1.dp)),
        color = color,
        shape = RoundedCornerShape(10.dp)
    ) {
        Text(
            modifier = Modifier.padding(20.dp),
            textAlign = TextAlign.Center,
            text = texto)
    }
}
```
## Box
- Permite apilar elementos uno encima del otro.
```kt
@Composable
fun EjemploBox() {
    Box(
        modifier = Modifier
            .size(200.dp)
            .background(Color.LightGray),  // Fondo de la Box
        contentAlignment = Alignment.Center // Alineación central de los elementos dentro de la Box
    ) {
        // Elemento de fondo
        Text(
            text = "Texto de Fondo",
            modifier = Modifier.align(Alignment.TopStart) // Alineado en la parte superior izquierda
        )

        // Elemento encima del texto
        Text(
            text = "Texto Encima",
            color = Color.White,
            modifier = Modifier.align(Alignment.Center) // Alineado en el centro de la Box
        )
    }
}
```

## Column y Row
- Column organiza sus elementos de manera vertical. 

```kt
@Composable
fun EjemploColumn() {
    Column(
        modifier = Modifier.padding(16.dp),   // Espaciado alrededor de la columna
        verticalArrangement = Arrangement.spacedBy(8.dp), // Espaciado entre elementos
        horizontalAlignment = Alignment.CenterHorizontally  // Alineación de los elementos en el centro
    ) {
        Text(text = "Elemento 1")
        Text(text = "Elemento 2")
        Text(text = "Elemento 3")
    }
}

```
- Row organiza sus elementos de manera horizontal.
```kt
@Composable
fun EjemploRow() {
    Row(
        modifier = Modifier.padding(16.dp),    // Espaciado alrededor de la fila
        horizontalArrangement = Arrangement.spacedBy(12.dp),  // Espaciado entre elementos
        verticalAlignment = Alignment.CenterVertically  // Alineación de los elementos en el centro vertical
    ) {
        Text(text = "Elemento 1")
        Text(text = "Elemento 2")
        Text(text = "Elemento 3")
    }
}
```
## FlowColumn y FlowRow
- FlowColumn organiza sus elementos de manera vertical, pero permite que los elementos se acomoden en varias "columnas" si no caben en una sola.
- FlowRow organiza sus elementos de manera horizontal, pero si no caben en una sola fila, los elementos se mueven a una nueva "fila" debajo de la anterior.

```kt
fun EjemploFlowRow() {
    FlowRow(
        modifier = Modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.spacedBy(8.dp),  // Espaciado entre los elementos
        verticalAlignment = Alignment.CenterVertically      // Alineación vertical en el centro
    ) {
        for (i in 1..20) {
            Button(onClick = {}) {
                Text("Elemento $i")
            }
        }
    }
}

@Composable
fun EjemploFlowColumn() {
    FlowColumn(
        modifier = Modifier.fillMaxHeight(),
        verticalArrangement = Arrangement.spacedBy(8.dp),  // Espaciado entre los elementos
        horizontalAlignment = Alignment.CenterHorizontally  // Alineación horizontal al centro
    ) {
        for (i in 1..20) {
            Button(onClick = {}) {
                Text("Elemento $i")
            }
        }
    }
}
```

## Imagenes
- Generalmente funcionará de la siguiente forma:
```kt
val painter = painterResource(id = R.drawable.balmis)
```
# MD3
## Ejemplo botón like
```kt
@Composable
private fun ButtonLikeBalmis(onClick: () -> Unit) {
    // Va a ser un botón con los colores de borde de Material Design 3, 
    // pero cambiando el contentColor a Rojo.
    val colors: ButtonColors = ButtonDefaults.outlinedButtonColors(
        contentColor = Color.Red
    )
    Button(
        onClick = onClick,
        // El bode tendrá el color del contentColor pero mantedrá 
        // el grosor definido de Material 3 para los botones con borde. 
        border = BorderStroke(
            width = ButtonDefaults.outlinedButtonBorder.width,
            color = colors.contentColor
        ),
        colors = colors,
        // El padding será el mismo que usen los botones con Icono.
        contentPadding = ButtonDefaults.ButtonWithIconContentPadding
    ) {
        // La imagen será un Icono de Material Design 3 pero ...
        Image(
            // Su tamaño será el de los iconos en los botones de Material 3.
            modifier = Modifier.size(ButtonDefaults.IconSize),
            // Icono añadido a los recursos
            painter = painterResource(id = R.drawable.favorite_24px),
            contentDescription = "Favorite",
            // El color del icono será también contentColor
            colorFilter = ColorFilter.tint(colors.contentColor)
        )
        // El espaciado tiene el mismo tamaño que el de los botones con Icono.
        Spacer(Modifier.size(ButtonDefaults.IconSpacing))
        Text("I love Balmis")
    }
}
```
## Checkbox
```kt
// Estará preparado para hacer State Hoisting por tanto es Stateless
@Composable
private fun CheckboxWithLabel(
    label: String,
    modifier: Modifier = Modifier,
    checkedState: Boolean,
    enabledState: Boolean = true,
    onStateChange: (Boolean) -> Unit) {
    // Definimos un Row para alinear Checkbox y Texto
    Row(
        modifier = modifier,
        verticalAlignment = Alignment.CenterVertically,
        horizontalArrangement = Arrangement.Start
    ) {
        Checkbox(
            checked = checkedState,
            onCheckedChange = onStateChange,
            enabled = enabledState,
        )
        Text(
            text = label,
            maxLines = 1,
            style = MaterialTheme.typography.bodySmall
        )
    }
}

@Preview(showBackground = true, name = "CheckBoxPreview")
@Composable
fun CheckBoxPreview() {
    var checkedState by remember { mutableStateOf(true) }
    HolaMundoTheme {
        Box {
            CheckboxWithLabel(
                label = "I Love Balmis",
                modifier = Modifier.padding(12.dp)
                                   .wrapContentWidth(),
                checkedState = checkedState,
                onStateChange = { checkedState = it }
            )
        }
    }
}
```

## Cards
- Son un contenedor, en el que la informacion está ordenada de una forma especifica, ejemplo
```kt
@Composable
private fun TarjetaBalmis(modifier: Modifier = Modifier) = ElevatedCard(
    modifier = modifier.then(Modifier.wrapContentSize()),
    elevation = CardDefaults.cardElevation(defaultElevation = 6.dp)
) {
    Column {
        Surface(
            modifier = Modifier.clip(CardDefaults.shape),
            color = MaterialTheme.colorScheme.primary
        ) {
            Image(
                modifier = Modifier.fillMaxWidth(),
                painter = painterResource(id = R.drawable.balmis),
                contentDescription = "IES Doctor Balmis",
                contentScale = ContentScale.FillWidth,
                alpha = 0.8f
            )
        }
        Spacer(modifier = Modifier.size(12.dp))
        Text(
            modifier = Modifier.padding(start = 12.dp, end = 12.dp),
            text = "IES Doctor Balmis",
            style = MaterialTheme.typography.headlineLarge
        )
        Text(
            modifier = Modifier.padding(start = 12.dp, end = 12.dp),
            text = "Alicante",
            style = MaterialTheme.typography.headlineSmall
        )
        Spacer(modifier = Modifier.size(12.dp))
        Text(
            modifier = Modifier.padding(start = 12.dp, end = 12.dp),
            text = "Instituto de Educación Secundaria donde se imparte el Ciclo Formativo"
                   + " de Grado Superior de Desarrollo de Aplicaciones Multiplataforma",
            style = MaterialTheme.typography.bodyMedium
        )
        Row(
            modifier = Modifier.fillMaxWidth().padding(12.dp),
            horizontalArrangement = Arrangement.End
        ) {
            Button(onClick = { }) {
                Text(text = "Saber más")
            }
        }
    }
}
```

## Textfiel
- Ejemplos:
```kt
@Composable
private fun OutlinedTextFieldWithErrorState(
    modifier: Modifier = Modifier,
    label: String,
    textoState: String,
    textoPista: String = "",
    leadingIcon: @Composable (() -> Unit)? = null,
    validacionState: Validacion,
    keyboardOptions: KeyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Text),
    keyboardActions: KeyboardActions = KeyboardActions(),
    onValueChange: (String) -> Unit
) {
    OutlinedTextField(
        modifier = modifier,
        value = textoState,
        onValueChange = onValueChange,
        singleLine = true,
        // Icono al principio del TextField por defecto vale null.
        leadingIcon = leadingIcon,
        // Como vamos a mostrar el texto de la pista o hint cuando estemos editando.
        // por defecto la pista es la cadena vacía.
        placeholder = {
            Text(
                text = textoPista,
                style = TextStyle(
                    color = MaterialTheme.colorScheme
                        .onSurfaceVariant.copy(alpha = 0.4f)
                )
            )
        },
        // Etiqueta personalizada que se muestra cuando no hay texto o
        // encima del TextField cuando estamos editando.
        // Le ponemos un asterisco si hay error como hemos especificado.
        label = { Text(if (validacionState.hayError) "${label}*" else label) },
        // La opciones del teclado permitirán la entrada alfanumérica.
        keyboardOptions = keyboardOptions,
        // Composable bajo el TextField que se muestra cuando hay error.
        supportingText = {
            if (validacionState.hayError) {
                Text(text = validacionState.mensajeError!!)
            }
        },
        // Parámetro con el estado del error.
        isError = validacionState.hayError,
        keyboardActions = keyboardActions
    )
}

@Composable
private fun OutlinedTextFieldEmail(
    modifier: Modifier = Modifier,
    label: String = "Email",
    emailState: String,
    validacionState: Validacion,
    onValueChange: (String) -> Unit
) {
    OutlinedTextFieldWithErrorState(
        modifier = modifier,
        label = label,
        textoState = emailState,
        // La pista cambiará.
        textoPista = "ejemplo@correo.com",
        // Las opciones de teclado serán para un email.
        keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
        // El icono será el de un email.
        leadingIcon = {
            Icon(
                imageVector = Icons.Filled.Email,
                contentDescription = "Email"
            )
        },
        // Será Stateles y la forma de validar la decidiremos al usarlo. 
        validacionState = validacionState,
        onValueChange = onValueChange
    )
}

@PreviewLightDark
@Composable
private fun TextFiledPreview() {
    var nombreState by remember { mutableStateOf("") }
    var validacionNombre by remember { mutableStateOf(object : Validacion {} as Validacion) }
    var emailState by remember { mutableStateOf("") }
    var validacionEmail by remember { mutableStateOf(object : Validacion {} as Validacion) }

    ProyectoBaseTheme {
        Surface {
        Column {
            OutlinedTextFieldWithErrorState(
                modifier = Modifier.fillMaxWidth(),
                label = "Nombre", textoState = nombreState,
                validacionState = validacionNombre,
                onValueChange = {
                    nombreState = it
                    validacionNombre = object : Validacion {
                        override val hayError: Boolean
                            get() = it.isEmpty()
                        override val mensajeError: String?
                            get() = "El nombre no puede estar vacío"
                    }
                }
            )
            OutlinedTextFieldEmail(
                modifier = Modifier.fillMaxWidth(), emailState = emailState,
                validacionState = validacionEmail,
                onValueChange = {
                    emailState = it
                    validacionEmail = object : Validacion {
                        override val hayError: Boolean
                            get() = it.isEmpty()
                                    || !Regex("^[A-Za-z](.*)([@]{1})(.{1,})(\\.)(.{1,})$")
                                .matches(it)
                        override val mensajeError: String?
                            get() = "El email no es válido"
                    }
                }
            )
        }
        }
    }
```

## Chip
- Es como un boton pero que varia de forma o visibilidad (parece ser)
```kt
@Composable
fun FilterChipWithIcon(
    modifier: Modifier = Modifier,
    seleccionadoState: Boolean = true,
    textoState: String = "Etiqueta",
    iconState: Painter? = null,
    onClick: () -> Unit = {}
) {
    FilterChip(
        modifier = modifier.then(Modifier.height(FilterChipDefaults.Height)),
        selected = seleccionadoState,
        onClick = onClick,
        label = { Text(textoState) },
        leadingIcon = {
            // Al estar seleccionado, mostramos el icono de selección
            // y sustituimos el icono iconState por este.
            if (seleccionadoState) {
                Icon(
                    painter = Filled.getCheckIcon(),
                    contentDescription = "Icono seleccionado",
                    modifier = Modifier.size(FilterChipDefaults.IconSize)
                )
            } else {
                iconState?.let {
                    Icon(
                        painter = it,
                        contentDescription = "Icono asociado a la etiqueta",
                        modifier = Modifier.size(FilterChipDefaults.IconSize)
                    )
                }
            }
        }
    )
}

@PreviewLightDark
@Composable
fun ChipPreview() {
    var filtrarPor2DAM by remember { mutableStateOf(false) }
    ProyectoBaseTheme {
        Surface (modifier = Modifier.background(MaterialTheme.colorScheme.surface)
                                    .padding(8.dp))
        {
            FilterChipWithIcon(
                seleccionadoState = filtrarPor2DAM,
                textoState = "Estudiante 2DAM",
                iconState = Filled.getPersonIcon(),
                onClick = { filtrarPor2DAM = !filtrarPor2DAM }
            )
        }
    }
}
```
## Dialogos
```kt
// Definimos una clase con los tres posibles contenidos del Box.
enum class ContenidoCaja { BOTON, DESPEDIDA_TRISTE, DESPEDIDA_ALEGRE }
// Composable Stateless que nos genera el composable con el contenido
// del Box según el estado gestionado por el AlertDialog
@Composable
fun ContenidoCaja(
    contenidoCajaState: ContenidoCaja,
    // Función que se ejecutará cuando pulsemos el botón de ver oferta.
    // y que cambiará el estado para mostrar el AlertDialog.
    onClickVerOferta: () -> Unit
) {
    when (contenidoCajaState) {
        ContenidoCaja.BOTON -> {
            Button(onClick = onClickVerOferta) {
                Text(text = "Ver Oferta")
            }
        }
        ContenidoCaja.DESPEDIDA_TRISTE -> {
            Text(text = "Adios, tu te lo pierdes")
        }
        ContenidoCaja.DESPEDIDA_ALEGRE -> {
            Text(text = "Enhorabuena, excelente elección")
        }
    }
}
@Composable
fun DialogoOferta(
    onAceptarDialogoOferta: () -> Unit,
    onRechazarDialogoOferta: () -> Unit,
    onCalcelaDialogoOferta: () -> Unit
) =
    // AlertDialog de Material 3
    AlertDialog(
        icon = {
            Icon(
                painterResource(R.drawable.gifts_24px),
                contentDescription = "Pregunta"
            )
        },
        title = { Text(text = "Oferta") },
        text = {
            // La lista la recordamos para no volver a generarla en cada recomposición.
            val ofertas = remember {
                listOf(
                    "Apartamento en Torrevieja (Alicante)",
                    "Aston Martin DB9",
                    "100 Ceniceros del IES Balmis"
                )
            }
            // Mostramos una oferta aleatoria de la lista.
            Text(text = ofertas.random())
        },
        // Lo que hacemos al pulsar fuera del AlertDialog para cancelarlo.
        onDismissRequest = onCalcelaDialogoOferta,
        confirmButton = {
            // En los AlertDialog de Material 3, el botón de confirmación
            // debería ser un TextButton
            TextButton(onClick = {
                // Acción de Aceptar la oferta.
                onAceptarDialogoOferta()
                onCalcelaDialogoOferta()
            }) {
                Text("Aceptar")
            }
        },
        dismissButton = {
            TextButton(onClick = {
                / Acción de Rechazar la oferta.
                onRechazarDialogoOferta()
                onCalcelaDialogoOferta()
            }) {
                Text("Rechazar")
            }
        }
    )
// Componente Box que implementará la lógica de la oferta.
@Composable
fun BoxOferta() = Box(
    modifier = Modifier
        .fillMaxWidth().size(height = 300.dp, width = 0.dp),
    contentAlignment = Alignment.Center
) {
    var verDialogoOferta by remember { mutableStateOf(false) }
    var contenidoCaja by remember { mutableStateOf(ContenidoCaja.BOTON) }

    ContenidoCaja(
        contenidoCajaState = contenidoCaja,
        onClickVerOferta = { verDialogoOferta = true }
    )

    if (verDialogoOferta) {
        DialogoOferta(
            onAceptarDialogoOferta = {
                contenidoCaja = ContenidoCaja.DESPEDIDA_ALEGRE
            },
            onRechazarDialogoOferta = {
                contenidoCaja = ContenidoCaja.DESPEDIDA_TRISTE
            },
        ) {
            verDialogoOferta = false
        }
    }
}
```





























