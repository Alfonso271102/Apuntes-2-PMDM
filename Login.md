- Composable
```kt
@Composable
fun UsuarioPassword(
    modifier: Modifier,
    loginState: String,
    validacionLogin: Validacion,
    passwordState: String,
    validacionPassword: Validacion,
    onValueChangeLogin: (String) -> Unit,
    onValueChangePassword: (String) -> Unit,
    onClickLogearse: () -> Unit
) {
    OutlinedTextFieldEmail(
        modifier = modifier,
        label = "Login",
        emailState = loginState,
        validacionState = validacionLogin,
        onValueChange = onValueChangeLogin
    )

    OutlinedTextFieldPassword(
        modifier = modifier,
        label = "Password",
        passwordState = passwordState,
        validacionState = validacionPassword,
        onValueChange = onValueChangePassword
    )


    Button(
        onClick = onClickLogearse,
        modifier = Modifier
            .fillMaxWidth()
    ) {
        Text("Login")
    }
}

```

- Screen del login
```kt

@Composable
fun LoginScreen(
    usuarioUiState: LoginUiState,
    validacionLoginUiState: ValidacionLoginUiState,
    onLoginEvent: (LoginEvent) -> Unit
) {
    val contexto = LocalContext.current
    var mostrarSnack by remember {
        mutableStateOf(false)
    };
    var mensaje by remember { mutableStateOf("") }
    val scope = rememberCoroutineScope()


    Box() {
        Column(
            verticalArrangement = Arrangement.Center,
            horizontalAlignment = Alignment.CenterHorizontally,
            modifier = Modifier.padding(20.dp)
        ) {


            UsuarioPassword(modifier = Modifier.fillMaxWidth(),
                loginState = usuarioUiState.login,
                passwordState = usuarioUiState.password,
                validacionLogin = validacionLoginUiState.validacionLogin,
                validacionPassword = validacionLoginUiState.validacionPassword,
                onValueChangeLogin = {
                    onLoginEvent(LoginEvent.LoginChanged(it))
                },
                onValueChangePassword = {
                    onLoginEvent(LoginEvent.PasswordChanged(it))
                },
                onClickLogearse = {

                    onLoginEvent(LoginEvent.OnClickLogearse)
                    mostrarSnack = true
                    scope.launch() {
                        delay(4000)
                        mostrarSnack = false
                    }
                })
            Spacer(modifier = Modifier.fillMaxHeight(0.1f))
        }
        if (mostrarSnack) {
            if (validacionLoginUiState.hayError)
                mensaje = validacionLoginUiState.mensajeError ?: ""
            else if (usuarioUiState.estaLogeado) mensaje =
                "Entrando a la APP con usuario ${usuarioUiState.login}"
            else mensaje = "Error, no existe el usuario o contraseña incorrecta"
            Snackbar(
                modifier = Modifier.align(Alignment.BottomCenter)
            ) {
                Text(text = mensaje)
            }
        }
    }
}

@Preview(showBackground = true)
@Composable
fun LoginScreenPreview() {
    val loginViewModel: LoginViewModel = viewModel()
    LoginTheme {
        // A surface container using the 'background' color from the theme
        Surface(
            modifier = Modifier.fillMaxSize(),
            color = MaterialTheme.colorScheme.background
        ) {

            LoginScreen(
                usuarioUiState = loginViewModel.usuarioUiState,
                validacionLoginUiState = loginViewModel.validacionLoginUiState,
                onLoginEvent = loginViewModel::onLoginEvent,
            )
        }
    }
}
```

- Eventos
```kt
sealed interface LoginEvent {
    data class LoginChanged(val login: String) : LoginEvent
    data class PasswordChanged(val password: String) : LoginEvent
    object  OnClickLogearse:LoginEvent
}
```
