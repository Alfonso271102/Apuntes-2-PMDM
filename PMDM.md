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
@Composable
fun ImagenEjemplo() {
    val painter = painterResource(id = R.drawable.balmis)
    Image(
        painter = painter,
        contentDescription = "Imagen de Balmis",
        modifier = Modifier
            .fillMaxWidth()
            .height(200.dp),
        contentScale = ContentScale.Crop // Ajusta cómo se escala la imagen
    )
}
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

# Hilt
- Cosas que hay que poner para el hilt:
- 1. Creamos la clase aplication en la raiz del proyecto
```kt
@HiltAndroidApp
class MiApplication : Application() {

}
```
- 2. En AndroidManifest.xml añadir:
```xml
<application
  android:name=".MiApplication" --> Añadir esto
  android:allowBackup="true"
  ...>

  ...

```

- 3. Añadir AndroidEntryPoint en mainActivity
```kt
@AndroidEntryPoint
class MainActivity : ComponentActivity() { ... }
```

- 4. Insertar HiltViewModel
```kt
@HiltViewModel
class EventoViewModel @Inject constructor(private val eventoRepository : EventoRepository) : ViewModel() {
```
- 5. Añadir hotviewmodel a Screen
```kt
@Composable
fun EventosScreen(viewModel: EventoViewModel = hiltViewModel()
```
- 6. Creamos el package "di" y dentro la clase AppModule
```kt
@Module
@InstallIn(SingletonComponent::class)
class AppModule {
    @Provides
    @Singleton
    fun provideSharedPreferences(@ApplicationContext context: Context) : SharedPreferences {
        return context.getSharedPreferences("miApp", Context.MODE_PRIVATE)
    }

    @Provides
    @Singleton
    fun provideEcentoDaoMock() : EventoDaoMock = EventoDaoMock()

    @Provides
    @Singleton
    fun provideMiRepositorio(eventoDaoMock : EventoDaoMock) : EventoRepository = EventoRepository(eventoDaoMock)
}
```
- 7. Creamos la inyección enel repositorio y en el DaoMock
```kt
class EventoRepository @Inject constructor(private val proveedorEventos:EventoDaoMock) {}
class EventoDaoMock @Inject constructor(){}
```

# Listas perezosas

- Ejemplo LazyColumn
```kt
@Composable
fun PedidoScreen(
    pedido: PedidoUiState,
) {

    LazyColumn(
       // columns = GridCells.Fixed(1),
        contentPadding = PaddingValues(all = 10.dp),
        verticalArrangement = Arrangement.spacedBy(4.dp)
    ) {
        items(pedido.articulos.size) { item ->
            CardPedido(
                articulo = pedido.articulos[item],
                modifier = Modifier.size(80.dp),

        )}
    }
}
```
- pedido.articulos.size ->  Representa el número total de elementos (artículos) que serán generados en la lista. 
- pedido.articulos[item]: Obtiene el artículo en la posición item de la lista articulos.

- Con datos indexados:
```kt
itemsIndexed(items = datosState) { i, dato ->
    Row(
        modifier = Modifier.clickable { onClickDato(i) },
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = "$i:", // Muestra el índice
            style = MaterialTheme.typography.titleMedium,
            modifier = Modifier.padding(8.dp).weight(0.2f)
        )
        TarjetaDato(
            modifier = Modifier.weight(0.8f),
            datoState = dato
        )
    }
}



//Ejemplo eliminando un elemento:

@Composable
fun DatosEnColumnaIndexada(
    datosState: MutableList<Datos>, // Cambia a MutableList para permitir modificaciones
    onClickDato: (Int) -> Unit = {}
) {
    LazyColumn(
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        itemsIndexed(items = datosState) { i, dato ->
            Row(
                modifier = Modifier
                    .clickable { 
                        datosState.removeAt(i) // Eliminar el elemento al pulsar
                    }
                    .padding(8.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                Text(
                    text = "$i:",
                    style = MaterialTheme.typography.titleMedium,
                    modifier = Modifier.weight(0.2f)
                )
                TarjetaDato(
                    modifier = Modifier.weight(0.8f),
                    datoState = dato
                )
            }
        }
    }
}
```

# Scaffold
## Ejemplo barra superior e inferior, con contenido en el centro
```kt
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraAplicacion(
    comportamientoAnteScroll: TopAppBarScrollBehavior
) = TopAppBar(
    title = {
        // El texto en TopAppBar solo puede tener una línea
        Text("", maxLines = 1, overflow = TextOverflow.Ellipsis)
    },
    navigationIcon = {
        IconButton(onClick = { }) {
            Icon(painter = painterResource(R.drawable.baseline_arrow_back_ios_24), contentDescription = null)
        }
    },
    actions = {
        IconButton(onClick = { }) {
            Icon(painter = painterResource(R.drawable.baseline_build_24), contentDescription = null)
        }
        IconButton(onClick = { }) {
            Icon(painter = painterResource(R.drawable.baseline_menu_24), contentDescription = null)
        }
    },
    scrollBehavior = comportamientoAnteScroll,
    colors = TopAppBarDefaults.topAppBarColors(
        containerColor = Color.LightGray
    )
)

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PantallaConScroll() {
    val comportamientoAnteScroll = TopAppBarDefaults.exitUntilCollapsedScrollBehavior()
    val comportamientoAnteScrollInf = BottomAppBarDefaults.exitAlwaysScrollBehavior()
    var itemSeleccionadoState: Int? by remember { mutableStateOf(null) }

    Scaffold(
        modifier = Modifier.nestedScroll(comportamientoAnteScroll.nestedScrollConnection),
        topBar = { BarraAplicacion(comportamientoAnteScroll) },
        bottomBar = {
            if (itemSeleccionadoState == null)
                BarraAppInferiorSeleccion(comportamientoAnteScrollInf)
            else
                BarraAppInferiorSeleccion(comportamientoAnteScrollInf)
        },
        content = { innerPadding ->
            ContenidoPrincipalScaffold(modifier = Modifier.padding(innerPadding))
        }
    )
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraAppInferiorSeleccion(
    comportamientoAnteScroll: BottomAppBarScrollBehavior
    = BottomAppBarDefaults.exitAlwaysScrollBehavior()
) {
    val descripcionEIconos = remember {
        listOf(
            "Eliminar Item" to R.drawable.baseline_edit_24,
            "Completar Item" to R.drawable.baseline_delete_24
        )
    }

    BottomAppBar(
        actions = {
            descripcionEIconos.forEach { (descripcion, icono) ->
                IconButton(
                    onClick = { }) {
                    Icon(
                        painter = painterResource(icono),
                        tint = MaterialTheme.colorScheme.secondary,
                        contentDescription = descripcion
                    )
                }
            }
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = {  },
                containerColor = BottomAppBarDefaults.bottomAppBarFabColor,
                contentColor = MaterialTheme.colorScheme.primary,
                elevation = FloatingActionButtonDefaults.bottomAppBarFabElevation()
            ) {
                Icon(painter = painterResource(R.drawable.baseline_add_24), "Localized description")
            }
        },
        scrollBehavior = comportamientoAnteScroll
    )
}


@Composable
fun ContenidoPrincipalScaffold(
    modifier: Modifier = Modifier
) {
    val colors = remember { listOf(Color(0xFF50A2E4), Color(0xFFFFFFFF)) }
    LazyColumn(modifier = modifier) {
        items(count = 25) {
            Box(
                Modifier
                    .fillMaxWidth()
                    .background(colors[it % colors.size])
            ) {
                Text(
                    text = "Item $it", modifier = Modifier.padding(16.dp)
                )
            }
        }
    }
}

```

## Ejemplo barra superior, navigation bar inferior y contenido en el centro con un lazygrid
```kt
@Composable
fun NavBar(
    indexScreenState: Int,
    onNavigateToScreen: (Int) -> Unit
) {
    val titlesAndIcons = remember {
        listOf(
            "Pantalla 1" to R.drawable.baseline_home_24,
            "Pantalla 2" to R.drawable.baseline_people_alt_24,
            "Pantalla 3" to R.drawable.baseline_add_alert_24
        )
    }

    NavigationBar {
        titlesAndIcons.forEachIndexed { index, (title, icon) ->
            NavigationBarItem(
                icon = { Icon(painter = painterResource(icon), contentDescription = title) },
                label = { Text(title) },
                selected = indexScreenState == index,
                onClick = { onNavigateToScreen(index) }
            )
        }
    }
}

@Composable
fun ContenidoPrincipalNavBar(
    indexScreenState: Int,
    modifier: Modifier = Modifier
) {
    val backgroundColor = when (indexScreenState) {
        0 -> MaterialTheme.colorScheme.primaryContainer
        1 -> MaterialTheme.colorScheme.secondaryContainer
        else -> MaterialTheme.colorScheme.tertiaryContainer
    }
    Box(
        modifier = modifier.then(
            Modifier
                .fillMaxSize()
                .background(color = backgroundColor)
        ),
        contentAlignment = Alignment.BottomCenter
    ) {


        LazyVerticalGrid(
            columns = GridCells.Fixed(2),
            modifier = Modifier
                .fillMaxSize()
                .padding(16.dp),
            horizontalArrangement = Arrangement.spacedBy(16.dp),
            verticalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            val elementos = listOf(
                "Pantalla 1" to R.drawable.baseline_accessibility_24,
                "Pantalla 2" to R.drawable.baseline_airplanemode_active_24,
                "Pantalla 3" to R.drawable.baseline_add_alert_24,
                "Pantalla 3" to R.drawable.baseline_3p_24
            )

            items(elementos) { (texto, icono) ->
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .aspectRatio(1f),
                    shape = RoundedCornerShape(16.dp),
                    colors = CardDefaults.cardColors(containerColor = Color(0xFFE8F1F8)),
                    elevation = CardDefaults.cardElevation(4.dp)
                ) {
                    Column(
                        modifier = Modifier
                            .fillMaxSize()
                            .padding(8.dp),
                        horizontalAlignment = Alignment.CenterHorizontally,
                        verticalArrangement = Arrangement.Center
                    ) {
                        Icon(
                            painter = painterResource(icono),
                            contentDescription = texto,
                            modifier = Modifier.size(48.dp),
                            tint = Color.Black
                        )
                        Spacer(modifier = Modifier.height(8.dp))
                        Text(
                            text = texto,
                            textAlign = TextAlign.Center
                        )
                    }
                }
            }
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PantallaNavBar() {
    var indexScreenState by remember { mutableStateOf(0) }
    val comportamientoAnteScroll = TopAppBarDefaults.pinnedScrollBehavior()
    Scaffold(
        modifier = Modifier.nestedScroll(comportamientoAnteScroll.nestedScrollConnection),
        topBar = { BarraAplicacion(comportamientoAnteScroll) },
        bottomBar = {
            NavBar(
                indexScreenState = indexScreenState,
                onNavigateToScreen = { indexScreenState = it }
            )
        },
        content = { innerPadding ->
            ContenidoPrincipalNavBar(
                indexScreenState = indexScreenState,
                modifier = Modifier.padding(innerPadding)
            )
        }
    )
}

@Preview(showBackground = true)
@Composable
fun PantallaNavBarPreview() {
    PantallaNavBar()
}
```

## Ejemplo barra superior (con opciones en las barras), navInferior, contenido central con imagen superior y pantalla con tabs
```kt
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun Tabs() {
    var tabIndexState by remember { mutableStateOf(0) }
    val titlesAndIcons = remember {
        listOf(
            "Todos" to R.drawable.baseline_home_24,
            "Pares" to R.drawable.baseline_people_alt_24,
            "Impares" to R.drawable.baseline_build_24
        )
    }
    PrimaryTabRow(selectedTabIndex = tabIndexState) {
        titlesAndIcons.forEachIndexed { index, (title, icon) ->
            Tab(
                selected = tabIndexState == index,
                onClick = { tabIndexState = index },
                text = { Text(
                    text = title,
                    maxLines = 2,
                    overflow = TextOverflow.Ellipsis)
                },
                icon = { Icon(painterResource(icon), contentDescription = null) }
            )
        }
    }
}

@Composable
fun ContenidoTabs(
    modifier: Modifier = Modifier
) {

    Column(modifier = modifier) {
        Image(contentDescription = "imagenarriba",
           painter = painterResource(R.drawable.obra),
            modifier = Modifier.fillMaxWidth().height(200.dp))
        Tabs()
        Column {
                Text(
                    text = "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras et lacus sed erat rutrum porta vel eget dolor. Pellentesque sed purus porta, vestibulum neque in, fringilla dolor. Quisque sollicitudin gravida iaculis. Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos. Proin sed lobortis ipsum. Etiam condimentum suscipit augue. Quisque porta nisi in porttitor varius. Etiam at tempus augue. Pellentesque dapibus lacus in leo cursus, at mattis ante cursus. Nullam vel sodales enim. Vestibulum ultrices scelerisque faucibus. Praesent urna est, scelerisque ac placerat nec, cursus et nisi. Sed non dolor nisi. Praesent feugiat nibh tortor.\n" +
                            "\n" +
                            "Interdum et malesuada fames ac ante ipsum primis in faucibus. Duis euismod vehicula facilisis. Vivamus gravida interdum est ac varius. Morbi in metus a sapien fringilla interdum interdum sed diam. Morbi sed hendrerit orci, id molestie ante. Suspendisse potenti. Vestibulum ultricies consequat nisl sed cursus. Nulla ex est, varius et sapien eget, consequat egestas risus. Nunc a sem nec erat euismod suscipit et et orci. Mauris vulputate dignissim enim, sed dignissim ex tristique sit amet. In a justo nec felis suscipit mollis. Aliquam mattis feugiat elit at malesuada. Cras dictum magna quis sem iaculis finibus. Vivamus rutrum viverra mollis. Vestibulum finibus auctor enim.",
                    modifier = Modifier.padding(16.dp).fillMaxWidth(),
                    textAlign = TextAlign.Center
                )
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PantallaConTabs() {
    val comportamientoAnteScroll = TopAppBarDefaults.exitUntilCollapsedScrollBehavior()
    var indexScreenState by remember { mutableStateOf(0) }

    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()

    var itemSeleccionadoState: Int? by remember { mutableStateOf(null) }
    val onSeleccionarItem: (Int) -> Unit = {
        itemSeleccionadoState = if (itemSeleccionadoState != it) it else null
        scope.launch {
            if (itemSeleccionadoState != null) {
                snackbarHostState.currentSnackbarData?.dismiss()
                snackbarHostState.showSnackbar(
                    message = "Saliendo de la aplicación",
                )
            }
        }
    }

    Scaffold(
        modifier = Modifier.nestedScroll(comportamientoAnteScroll.nestedScrollConnection),
        topBar = { BarraAplicacionDropMenu(false, comportamientoAnteScroll, snackbarHostState) },
        content = { innerPadding ->
            ContenidoTabs(modifier = Modifier.padding(innerPadding))
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = {
                    scope.launch {
                        if (itemSeleccionadoState != null) {
                            snackbarHostState.currentSnackbarData?.dismiss()
                            snackbarHostState.showSnackbar(
                                message = "Item $itemSeleccionadoState borrado",
                                withDismissAction = true,
                                duration = SnackbarDuration.Indefinite
                            )
                        }
                    }
                }
            ) {
                Icon(imageVector = Icons.Filled.Delete, contentDescription = null)
            }
        },
        bottomBar = {  NavBar(
            indexScreenState = indexScreenState,
            onNavigateToScreen = { indexScreenState = it }
        )}
    )
}

@Immutable
data class ItemMenuDesplegable(
    val descripcion: String,
    val onClick: () -> Unit
)

@Composable
fun AccionesConMenuDesplegable(
    itemsMenu : List<ItemMenuDesplegable>
) {
    // Precondición de uso
    if (itemsMenu.count() < 2)
        throw IllegalArgumentException("Se requieren al menos 2 items en el menú desplegable")
    var expandidoState by remember { mutableStateOf(false) }
    val cerrarMenu: () -> Unit = { expandidoState = false }

    IconButton(onClick = itemsMenu[0].onClick) {
        Icon(
            painter = painterResource(R.drawable.baseline_build_24),
            contentDescription = itemsMenu[0].descripcion
        )
    }
    IconButton(onClick = { expandidoState = true }) {
        Icon(painter = painterResource(R.drawable.baseline_menu_24), contentDescription = null)
    }

    DropdownMenu(
        expanded = expandidoState,
        onDismissRequest = cerrarMenu
    ) {
        for (i in 0 until itemsMenu.count()) {
            DropdownMenuItem(
                text = { Text(itemsMenu[i].descripcion) },
                onClick = {
                    itemsMenu[i].onClick()
                    cerrarMenu()
                },
                leadingIcon = {
                    Icon(
                        painter = painterResource(R.drawable.baseline_menu_24),
                        contentDescription = itemsMenu[i].descripcion
                    )
                })
        }
    }
}



@Composable
fun AccionesConMenuDesplegableSinSeleccion(snackbarHostState: SnackbarHostState ) {
    val scope = rememberCoroutineScope()

    val descripcionEIconos = remember {
        listOf(
            ItemMenuDesplegable(
                descripcion = "Cerrar sesión", onClick = {
                    scope.launch {
                        snackbarHostState.currentSnackbarData?.dismiss()
                        snackbarHostState.showSnackbar("Se está cerrando la sesión")
                    }
                }
            ),
            ItemMenuDesplegable(
                descripcion = "Salir aplicación", onClick = {
                    scope.launch {
                        snackbarHostState.currentSnackbarData?.dismiss()
                        snackbarHostState.showSnackbar("Saliendo de la aplicación")
                    }
                }
            ),
        )
    }
    return AccionesConMenuDesplegable(itemsMenu = descripcionEIconos)
}




@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraAplicacionDropMenu(
    itemSeleccionadoState: Boolean,
    comportamientoAnteScroll: TopAppBarScrollBehavior,
    snackbarHostState: SnackbarHostState
) = TopAppBar(
    title = {
        Text("", maxLines = 1, overflow = TextOverflow.Ellipsis)
    },
    navigationIcon = {
        IconButton(onClick = { }) {
            Icon(painter = painterResource(R.drawable.baseline_arrow_back_ios_24), contentDescription = null)
        }
    },
    actions = {
        if (!itemSeleccionadoState) AccionesConMenuDesplegableSinSeleccion(snackbarHostState)
    },
    scrollBehavior = comportamientoAnteScroll,
    colors = TopAppBarDefaults.topAppBarColors(
        containerColor = Color.LightGray
    )
)


@Preview(showBackground = true)
@Composable
fun PantallaConTabsPreview() {
    PantallaConTabs()
}
```

## Ejemplo barra lateral izquierda
```kt
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraAppEnScaffoldDentroNavDrawer(
    onClickActionMenu: () -> Unit,
    comportamientoAnteScroll: TopAppBarScrollBehavior
) = TopAppBar(
    title = {
        Text("", maxLines = 1, overflow = TextOverflow.Ellipsis)
    },
    navigationIcon = {
        IconButton(onClick = onClickActionMenu) {
            Icon(painter = painterResource(R.drawable.baseline_arrow_back_ios_24), contentDescription = null)
        }
    },
    actions = {
        IconButton(onClick = { }) {
            Icon(painter = painterResource(R.drawable.baseline_build_24), contentDescription = null)
        }
        IconButton(onClick = { }) {
            Icon(painter = painterResource(R.drawable.baseline_menu_24), contentDescription = null)
        }
    },
    scrollBehavior = comportamientoAnteScroll
)

@Composable
fun PantallaPrincipalScaffoldDentroNavDrawer(
    selecteItemState: ItemMenuEjemploNavDrawer,
    modifier: Modifier = Modifier
) {
    val backgroundColor = when (selecteItemState.index) {
        0 -> MaterialTheme.colorScheme.primaryContainer
        1 -> MaterialTheme.colorScheme.secondaryContainer
        else -> MaterialTheme.colorScheme.tertiaryContainer
    }
    Box(
        modifier = modifier.then(
            Modifier
                .fillMaxSize()
                .background(color = backgroundColor)
        ),
        contentAlignment = Alignment.TopCenter
    ) {
        Text(
            modifier = Modifier.padding(top = 32.dp),
            text = selecteItemState.nombre,
            textAlign = TextAlign.Center,
            style = MaterialTheme.typography.headlineLarge
        )
    }
}

enum class ItemMenuEjemploNavDrawer(
    val index: Int,
    val nombre: String
) {
    Pantalla1(index = 0,nombre = "Amigos"),
    Pantalla2(index = 1, nombre = "Coro"),
    Pantalla3(index = 2, nombre = "Familia"),
    Pantalla4(index = 3, nombre = "Trabajo"),
    Pantalla5(index = 4, nombre = "Vecinos")
}

@Composable
fun ContenidoNavDrawer(
    selecteItemState: ItemMenuEjemploNavDrawer,
    onItemSelected: (ItemMenuEjemploNavDrawer) -> Unit,
    modifier: Modifier = Modifier
) {
    val items = remember {
        listOf(
            ItemMenuEjemploNavDrawer.Pantalla1,
            ItemMenuEjemploNavDrawer.Pantalla2,
            ItemMenuEjemploNavDrawer.Pantalla3,
            ItemMenuEjemploNavDrawer.Pantalla4,
            ItemMenuEjemploNavDrawer.Pantalla5
        )
    }
    ModalDrawerSheet(modifier = modifier) {
        Image(contentDescription = "imagenarriba",
            painter = painterResource(R.drawable.amigos),
            modifier = Modifier.fillMaxWidth().height(200.dp))
        Spacer(Modifier.height(12.dp))
        items.forEach { item ->
            NavigationDrawerItem(
                label = { Text(item.nombre) },
                selected = item.index == selecteItemState.index,
                onClick = { onItemSelected(item) },
                modifier = Modifier.padding(NavigationDrawerItemDefaults.ItemPadding)
            )
        }
    }
}

@Composable
fun PantallaConNavDrawer() {
    val drawerState = rememberDrawerState(DrawerValue.Closed)
    var selectedItem by remember { mutableStateOf(ItemMenuEjemploNavDrawer.Pantalla1) }
    val scope = rememberCoroutineScope()
    val onItemSelected: (ItemMenuEjemploNavDrawer) -> Unit = {
        scope.launch { drawerState.close() }
        selectedItem = it
    }
    val onClickActionMenu: () -> Unit = {
        scope.launch { drawerState.open() }
    }
    ModalNavigationDrawer(
        drawerState = drawerState,
        drawerContent = {
            ContenidoNavDrawer(
                selecteItemState = selectedItem,
                onItemSelected = onItemSelected
            )
        },
        content = {
            ScaffoldDentroNavDrawer(
                selecteItemState = selectedItem,
                onClickActionMenu = onClickActionMenu,
            )
        }
    )
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ScaffoldDentroNavDrawer(
    selecteItemState: ItemMenuEjemploNavDrawer,
    onClickActionMenu: () -> Unit,

) {
    var indexScreenState by remember { mutableStateOf(0) }
    val comportamientoAnteScroll = TopAppBarDefaults.pinnedScrollBehavior()
    Scaffold(
        modifier = Modifier.nestedScroll(comportamientoAnteScroll.nestedScrollConnection),
        topBar = {
            BarraAppEnScaffoldDentroNavDrawer(
                onClickActionMenu = onClickActionMenu,
                comportamientoAnteScroll = comportamientoAnteScroll
            )
        },
        bottomBar = {
            NavBar(
                indexScreenState = indexScreenState,
                onNavigateToScreen = { indexScreenState = it }
            )
        },
        content = { innerPadding ->
            PantallaPrincipalScaffoldDentroNavDrawer(
                selecteItemState = selecteItemState,
                modifier = Modifier.padding(innerPadding)
            )
        }
    )
}

@Preview(showBackground = true)
@Composable
fun PantallaConNavDrawerPreview() {
            PantallaConNavDrawer()
}
```

## Navegacion

- Vreamos el paquete nav dentro de ui
- Dentro creamos el NavHost
```kt
@Composable
fun MiNavHost(
    navController: NavHostController,
    onClickActionMenu: () -> Unit,
    selectedItem: ItemMenuEjemploNavDrawer,
    onSalir: () -> Unit
){
    val vmListaContactos = hiltViewModel<ListadoContactosViewModel>()
    val vmTrabajadors = hiltViewModel<TrabajoViewModel>()

    NavHost(
        navController = navController,
        startDestination = PantallaListaContactosRoute // Pantalla inicial por defecto
    ){
        pantallaListaContactosDestination(
            onNavigatePantallaTrabajadores = { contacto (dato que envio a la otra pantalla) ->
                navController.navigate(PantallaTrabajadoresRoute(contacto))
            },
            selecteItemState = selectedItem,
            onClickActionMenu = onClickActionMenu,
            onSalir = onSalir,
            contactos =  vmListaContactos.lista.toContactoUiState()
        )
        pantallaTrabajadoresDestination(
            onNavegarAtras = {
                navController.popBackStack() ; // navegar atras
                 onClickActionMenu()
            },
            onSalir = onSalir,
            viewModel = vmTrabajadors
        )
    }

}

```

- Creamos un route para cada pantalla

```kt
@Serializable
object PantallaListaContactosRoute //(object porque no recibe parametros, sino data class)

fun NavGraphBuilder.pantallaListaContactosDestination(
    selecteItemState: ItemMenuEjemploNavDrawer,
    onClickActionMenu: () -> Unit,
    onSalir: () -> Unit,
    contactos: List<ContactoUiState>,
    onNavigatePantallaTrabajadores: (Int) -> Unit
){
    composable<PantallaListaContactosRoute> { backStackEntry ->
        ScaffoldDentroNavDrawer(
            selecteItemState = selecteItemState,
            onClickActionMenu = onClickActionMenu,
            onSalir = onSalir,
            contactos = contactos,
            onNavigatePantallaTrabajadores = onNavigatePantallaTrabajadores
        )
    }
}
```

```kt
@Serializable
data class PantallaTrabajadoresRoute(val contacto: Int) // dato que recibo

fun NavGraphBuilder.pantallaTrabajadoresDestination(
    onNavegarAtras: () -> Unit,
    onSalir: () -> Unit,
    viewModel: TrabajoViewModel
){
    composable<PantallaTrabajadoresRoute> { backStackEntry -> //vuelve atras en la pila
        val datos : PantallaTrabajadoresRoute = remember{ backStackEntry.toRoute<PantallaTrabajadoresRoute>()} (recupera todos los datos cargados desde la otra pantalla para que podamos acceder a ellos)

        PantallaConTabs(
            contacto = datos.contacto, (acceso a los datos de la otra pantalla)
            onSalir = onSalir,
            onNavegarAtras = onNavegarAtras,
            viewModel = viewModel
        )
    }
}
```

- Podemos por ejemplo poner el navhost en un content de un nav draver, para conseguir que se navegue, por ejemplo:

```kt
@Composable
fun PantallaConNavDrawer(viewModelListado: ListadoContactosViewModel
) {
    val navController = rememberNavController() //(lo definimos)


    val drawerState = rememberDrawerState(DrawerValue.Closed)
    var selectedItem by remember { mutableStateOf(ItemMenuEjemploNavDrawer.Pantalla1) }
    val scope = rememberCoroutineScope()

    val activity = LocalContext.current as? Activity

    val onItemSelected: (ItemMenuEjemploNavDrawer) -> Unit = {
            item ->
        scope.launch { drawerState.close() }
        selectedItem = item
        viewModelListado.onItemSeleccionado(item.index)
    }
    val onClickActionMenu: () -> Unit = {
        scope.launch { drawerState.open() }
    }
    ModalNavigationDrawer(
        drawerState = drawerState,
        drawerContent = {
            ContenidoNavDrawer(
                selecteItemState = selectedItem,
                onItemSelected = onItemSelected,
            )
        },
        content = {
            MiNavHost(
                navController = navController, //(aqui lo cargamos)
                onClickActionMenu = onClickActionMenu,
                selectedItem = selectedItem,
                onSalir = {activity?.finish()}
            )
        }
    )
}

``` 

# Room

## PreAjustes:
- lib.versions.toml deberemos comprobar que hemos definido tener:
```js
[versions]
kotlin = "2.0.20"
ksp = "2.0.20-1.0.25"
room = "2.6.1"

[libraries]
androidx-room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
androidx-room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }

[plugins]
devtools-ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```
- build.gradle.kts raíz del proyecto añadiremos el siguiente plugin:
```js
plugins {
    alias(libs.plugins.devtools.ksp) apply false
}
```
- build.gradle.kts del módulo de la aplicación (app) añadiremos:
```js
plugins {
    alias(libs.plugins.devtools.ksp)
}
...
dependencies {
    implementation(libs.androidx.room.ktx)
    ksp(libs.androidx.room.compiler)
}
```

## Crear las entidades que definen nuestro modelo
```kt
@Entity(tableName = "clientes" // en caso de que queramos que la tabla se llame diferente en la BD)
data class ClienteEntity(
    @PrimaryKey // si queremos definir una clave primaria
    @ColumnInfo(name = "dni")
    val dni: String,
    @ColumnInfo(name = "nombre")
    val nombre: String,
    @ColumnInfo(name = "apellidos"))
```

- Para referenciar a una clave primaria compuesta por más de un campo, tendríamos que indicarlo con la propiedad primaryKeys de la anotación @Entity. De igual manera lo haríamos en el caso de necesitar una clave ajena, pero en este caso con la propiedad foreignKeys.
```kt
// Expresaríamos que un pedido define una clave ajena dni en clientes.
@Entity(tableName = "pedidos",
    foreignKeys = arrayOf(
        ForeignKey(
            entity = ClienteEntity::class,
            parentColumns = arrayOf("dni"),
            // Nombre de la columna en la tabla pedidos
            childColumns = arrayOf("dni_cliente"),
            onDelete = CASCADE
        )
    )
)
data class PedidoEntity(...)

```

- Embeber campos con referencias a otras tablas:
```kt
data class Direccion(
    val calle: String?,
    val ciudad: String?,
    val pais: String?,
    @ColumnInfo(name = "codigo_postal") 
    val codigoPostal: String?
)

@Entity(tableName = "clientes")
data class ClienteEntity(
    @PrimaryKey
    @ColumnInfo(name = "dni")
    val dni: String,
    @ColumnInfo(name = "nombre")
    val nombre: String,
    @ColumnInfo(name = "apellidos")
    val apellidos: String,
    // Marcamos como embebe @Embedded el campo a descomponer en la tabla
    @Embedded val direccion: Direccion?
)
```
- Definiremos primero una clase denominada RoomConverters dentro del paquete .data.room con todos los métodos convertidores. Ojo, no importa el nombre del método, sino su signatura (parámetro de entrada y de salida). Por ejemplo para fecha podría ser:
```kt
// Estamos presuponiendo que nuestras fechas nunca son nulas.
class RoomConverters {
    @TypeConverter
    fun fromTimestamp(value: Int): LocalDate {
        return LocalDate.fromEpochDays(value)
    }

    @TypeConverter
    fun dateToTimestamp(date: LocalDate): Int {
        return date.toEpochDays()
    }
}
```

## Ejemplo completo
-  Clase PedidoEntity dentro del paquete .data.room
```kt
@Entity(tableName = "pedidos",
    foreignKeys = arrayOf(
        ForeignKey(
            entity = ClienteEntity::class,
            parentColumns = arrayOf("dni"),
            childColumns = arrayOf("dni_cliente"),
            onDelete = CASCADE
        )
    )
)
data class PedidoEntity(
    // El id será autogenerado insertando un 0.
    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "id")
    val id: Int,

    @ColumnInfo(name = "dni_cliente")
    val dniCliente: Int,

    // Indicamos que siempre debe tener una fecha.
    // Esta en la DB se guardará como un Int.
    @NonNull
    @ColumnInfo(name = "fecha")
    val fecha: LocalDate
)
```

## Definiendo relaciones entre Objetos

```kt
data class ClienteConPedidos(
    // Sabe recuperar el objeto embebido.
    @Embedded val cliente: ClienteEntity,
    @Relation(
          parentColumn = "dni",
          // Nombre de la columna en la parte del muchos
          entityColumn = "dni_cliente"
    )
    val pedidos: List<PedidoEntity>
)
```

## Crear los Objetos de Acceso a la Base de Datos
- Creacion de DAO con metodos de conveniencia
```kt
@Dao
interface ClienteDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(cliente : ClienteEntity)

    @Delete
    suspend fun delete(cliente : ClienteEntity)

    @Update
    suspend fun update(cliente : ClienteEntity)

    // Si no pasamos la entidad y lo que tenemos 
    // es la PK deberemos hacer una consulta con @Query
    // pasando la PK como parámetro.
    @Query("DELETE FROM clientes WHERE dni = :dni")
    suspend fun deleteByDni(dni: String)
}
```
- Creacion de DAO con metodos de búsqueda
```kt
@Dao
interface ClienteDao {
    // ... aquí van los métodos de conveniencia CRUD

    @Query("SELECT * FROM clientes")
    suspend fun get(): List<ClienteEntity>

    @Query("SELECT * FROM clientes WHERE dni IN (:dni)")
    suspend fun getFromDni(dni : String): ClienteEntity

    @Query("SELECT COUNT(*) FROM clientes")
    suspend fun count(): Int
}
```
- Selección de un subconjunto de columnas
- si solo necesitamos una informacion determinada, podemos crear un DTO
```kt
data class NombreApellidosDTO(
    @ColumnInfo(name = "nombre")
    val nombre: String,
    @ColumnInfo(name = "apellidos")
    val apellidos: String
)


// Y solamente se tendrá que indicar en el Dao que el método correspondiente devolverá los datos de este tipo:

@Query("SELECT nombre, apellidos FROM clientes")
suspend fun getNombreApellido(): List<NombreApellidosDTO>
```

## Crear la Base de Datos Room y consumirla
- Para crear la base de datos a partir de las entidades definidas y las operaciones que se realizarán sobre estas a partir de los DAOs de nuestra app:
```kt
@Database(
    entities = [ClienteEntity::class, PedidoEntity::class],
    version = 1
)
@TypeConverters(RoomConverters::class)
abstract class TiendaDB : RoomDatabase() {
    abstract fun clienteDao() : ClienteDao
    abstract fun pedidoDao() : PedidoDao

    companion object {
        fun getDatabase(
            context: Context
        ) = Room.databaseBuilder(
            context,
            TiendaDB::class.java, "tienda"
        )
        .allowMainThreadQueries()
        .fallbackToDestructiveMigration()
        .build()
    }
}

```
## Preparando la Base de Datos y los DAOs con Hilt

- Para inyectar la BD definimos provideTiendaDatabase que llama al método estático TiendaDB.getDatabase(context) para instanciarla. Fíjate que el contexto de la aplicación se inyecta usando la anotación @ApplicationContext de Hilt.
```kt
@Provides
@Singleton
fun provideTiendaDatabase(
    @ApplicationContext context: Context
): TiendaDB = TiendaDB.getDatabase(context)
```

- Para inyectar los DAOs, definimos provideClienteDao y providePedidoDao que inyectan los DAOs de la BD. Fíjate que se inyecta la BD y se llama a los métodos abstractos que devuelven los DAOs.
```kt
@Provides
@Singleton
fun provideClienteDao(
  tiendaDB: TiendaDB
): ClienteDao = tiendaDB.clienteDao()
@Provides
@Singleton
fun providePedidoDao(
  tiendaDB: TiendaDB
): PedidoDao = tiendaDB.pedidoDao()
```
## Inyectando los DAOs en los repositorios
```kt
class ClienteRepository @Inject constructor(
    private val clienteDao: ClienteDao
) {
    ...
}
```

## Cargando datos de prueba en la BD
- En el MyApplication
```kt
@HiltAndroidApp
class MyApplication : Application() {
@Inject
    lateinit var daoClientes: ClienteDao

    override fun onCreate() {
        super.onCreate()

        runBlocking {
            if (daoClientes.count() == 0) {
                daoClientes.insert(
                    ClienteEntity("12345678A", "Juan", "Pérez"))
                daoClientes.insert(
                    ClienteEntity("87654321B", "María", "García"))
                    
        }
    }
}
```





































