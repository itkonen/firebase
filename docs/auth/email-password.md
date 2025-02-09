# Email & Password

As shown in the [Prebuilt UI page](/guide/ui.md) one can allow users to sign in with an email and a password. A we detail here we can also do so by building the UI ourselves, which very straightforward using shiny's inputs. However, while the prebuilt UI does not distinguish between account creation and signing in, we will here.

We will thus build an application that, using an email and password, allows the user to create an account then login with said account. Let us first build the skeleton of the application. 

```r
library(shiny)
library(firebase)

# modals
register <- modalDialog(
  title = "Register",
  textInput("email_create", "Your email"),
  passwordInput("password_create", "Your password"),
  footer = actionButton("create", "Register")
)

sign_in <- modalDialog(
  title = "Sign in",
  textInput("email_signin", "Your email"),
  passwordInput("password_signin", "Your password"),
  actionButton("signin", "Sign in")
)

ui <- fluidPage(
  useFirebase(), # import dependencies
  actionButton("register_modal", "Register"),
  actionButton("signin_modal", "Signin")
)

server <- function(input, output){

  # open modals
  observeEvent(input$register_modal, {
    showModal(register)
  })

  observeEvent(input$signin_modal, {
    showModal(sign_in)
  })

}

shinyApp(ui, server)
```

Now with this skeleton ready let's initialise firebase and implement the account creation. 

First, initialise the authentication with `FirebaseEmailPassword`. Then use the `create` method to create the user's account using their email and password. One thing in the snippet below that is optional but very nice to have is to observe the `get_created` method. This method returns information on the account creation; a `list` that contains the object `success`, a boolean indicating whether the authentication was successful as well as the `response` from Firebase. Observing for changes in the account creation lets us show a notification to the user and remove the modal when necessary.

```r
library(shiny)
library(firebase)

# modals
register <- modalDialog(
  title = "Register",
  textInput("email_create", "Your email"),
  passwordInput("password_create", "Your password"),
  footer = actionButton("create", "Register")
)

sign_in <- modalDialog(
  title = "Sign in",
  textInput("email_signin", "Your email"),
  passwordInput("password_signin", "Your password"),
  footer = actionButton("signin", "Sign in")
)

ui <- fluidPage(
  useFirebase(), # import dependencies
  actionButton("register_modal", "Register"),
  actionButton("signin_modal", "Signin")
)

server <- function(input, output){

  f <- FirebaseEmailPassword$new()

  # open modals
  observeEvent(input$register_modal, {
    showModal(register)
  })

  observeEvent(input$signin_modal, {
    showModal(sign_in)
  })

  # create the user
  observeEvent(input$create, {
    f$create(input$email_create, input$password_create)
  })

  # check if creation sucessful
  observeEvent(f$get_created(), {
    created <- f$get_created()
    
    if(created$success){
      removeModal()
      showNotification("Account created!", type = "message")
    } else {
      showNotification("Error!", type = "error")
    }

    # print results to the console
    print(created)
  })

}

shinyApp(ui, server)
```

The sign in process is like that of other authentication methods.

```r
library(shiny)
library(firebase)

# modals
register <- modalDialog(
  title = "Register",
  textInput("email_create", "Your email"),
  passwordInput("password_create", "Your password"),
  footer = actionButton("create", "Register")
)

sign_in <- modalDialog(
  title = "Sign in",
  textInput("email_signin", "Your email"),
  passwordInput("password_signin", "Your password"),
  footer = actionButton("signin", "Sign in")
)

ui <- fluidPage(
  useFirebase(), # import dependencies
  actionButton("register_modal", "Register"),
  actionButton("signin_modal", "Signin"),
  plotOutput("plot")
)

server <- function(input, output){

  f <- FirebaseEmailPassword$new()

  # open modals
  observeEvent(input$register_modal, {
    showModal(register)
  })

  observeEvent(input$signin_modal, {
    showModal(sign_in)
  })

  # create the user
  observeEvent(input$create, {
    f$create(input$email_create, input$password_create)
  })

  # check if creation sucessful
  observeEvent(f$get_created(), {
    created <- f$get_created()
    
    if(created$success){
      removeModal()
      showNotification("Account created!", type = "message")
    } else {
      showNotification("Error!", type = "error")
    }

    # print results to the console
    print(created)
  })

  observeEvent(input$signin, {
    removeModal()
    f$sign_in(input$email_signin, input$password_signin)
  })

  output$plot <- renderPlot({
    f$req_sign_in()
    plot(cars)
  })

}

shinyApp(ui, server)
```

![](firebase_email_password.gif)
