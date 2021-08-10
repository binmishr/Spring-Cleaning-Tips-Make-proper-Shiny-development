# Spring-Cleaning-Tips-Make-proper-Shiny-development

1. Deep breeze and allocate some time

Do not avoid spring cleaning simply because you don’t know where to start from. Prioritize some time for the task and get inspired by our following points.

2. Get things organized = Put your app in a package

Before starting to clean, usually you must think how you want to get organized and get some equipment. E.g. initially, you need a cupboard with several compartments to simplify setting up a methodical storage.

Why not use golem? This package provides a framework with a series of tools for building robust shiny applications and set them up for production. Initialize your new golem package via RStudio GUI:

golem

Sort your code into the right place:

    under the R folder: split UI and Server code into functions and add here any other functions you may have;
    under inst/app/www: add your external dependencies like images and CSS, but do not forget to tell your app where to find them (check golem::golem_add_external_resources())

Fill the DESCRIPTION (golem helps with that too):

#' Fill golem description file
golem::fill_desc(
  # The Name of the package containing the App 
  pkg_name = "shiny_spring_box",
  # The Title of the package containing the App 
  pkg_title = "The_Shiny_Spring_Box",
  # The Description of the package containing the App
  pkg_description = "A well sorted and sustainable Shiny App packaged with golem",
  # Your First Name
  author_first_name = "John",
  # Your Last Name
  author_last_name = "Clean",
  # Your Email
  author_email = "johnclean@springbox.com",
) 

Now you are ready to launch your newly packaged app: run_app().

3. Sort your code systematically, throw away what you don’t need, make the rest available in a handy place.

Think of modules as nice boxes that you use to organize your stuff in the cupboard. It looks tidy and if well labeled it makes things very easy to find. Modularizing your Shiny App with modules will bring you 3 big advantages:

    Smaller chunks of code are easier to maintain and to debug, and they improve the overall readability.
    Modules can be reused (even outside the app), which helps remove duplicity, making the app slim and consistent.
    Modules deal with the ‘id’ namespacing constraints better than you could.

Be aware that, since shiny version >= 1.5.0, the syntax used to call the server side of a module has changed, and the use of modelserver() is not required anymore.

Take a look at the example of a simple app using modules:

# module ui
cleaninghero_ui <- function(id) {
  ns <- NS(id)
  tagList(
    textInput(ns("inp_name"), label = NULL,
                   placeholder = "your cleaning specialist"),
    actionButton(ns("button_validate"), label = "Validate"),
    hr(),
    textOutput(ns("confirmation"))
  )
}

# module server
cleaninghero_server <- function(id, default_who) {
  moduleServer(
    id,
    function(input, output, session) {
      who <- reactiveVal("Mr Clean")
      # when the "Validate" button is clicked
      observeEvent(input$button_validate, {
        who(input$inp_name)
        # confirm Mr. Clean if there is no-one to validate
        if (input$inp_name == "") {
          who("Mr Clean")
        }
      })
      # confirm selection
      output$confirmation <- renderText({
        paste0("The cleaning specialist ", who(),
                    " will help you sort things out!")
      })
    }
  )
}


# app
ui <- fluidPage(
  titlePanel("Using Shiny Modules"),
  verticalLayout(
    cleaninghero_ui(id = "choice_1"),
    sliderInput(inputId = "slider_inp_example", 
                label = "Evaluate your cleaning hero level of Stars",
                value = 3, min = 0, max = 5),
    actionButton(inputId = "resetbnt", label = "Reset Stars")
  )
)
server <- function(input, output, session) {
  cleaninghero_server(id = "choice_1")
  #Update widgets
  observeEvent(input$resetbnt, {
    updateSliderInput(session, "slider_inp_example", value = 3)
  })
}

shinyApp(ui, server)

Organize the core functionalities of your app into modules with the help of golem::add_modules().

4. Clean even deeper! Extract the business logic into separate functions.

This step has the same purpose as the one above, and will result in more clarity, re-usability, avoiding redundancy and enhancing consistency. Here we are talking about computations, data preparation, help functions or anything which can be run outside an interactive context. Again golem gives you a hand, build your functions with golem::add_utils() and golem::add_fct().

5. Make this spring action sustainable, keep an app working and shining in the long term.

Testing is the magic word! Only through it you can have some guarantees that your app will continue working as expected. However, Shiny App definitively brings some testing challenges, since ideally you should test:

    function and computations,
    app visuals,
    reactivity,
    ‘JavaScript’ components,
    performance.

-> golem, as a test automation tool and framework, will support your unit tests. To create the testing folder structure, run golem::use_recommended_tests().

-> With testServer(), a sort of simulation of your app, you can test reactive code from the server side of the modules. Careful here, a bunch of functions like update*(), showModal() or insertUI() uses JavaScript behind the scenes which is not get covered by testServer(), which only tests the server function and ignores completely the ui components.

-> To test those parts using JavaScript, wrap app <- ShinyDriver$new() initializing and interacting with app$setInputs() to set values of input widgets, and app$getValue() to retrieve values into a test_that()call.

# ---- shiny-test-javascript
context("app-function")

test_that("Javascript is included and works", {
  app <- shinytest::ShinyDriver$new("../app/")
  # set the stars 3
  app$setInputs(slider_inp_example = 2)
  expect_equal(app$getValue("slider_inp_example"), 2)
  # clicking reset button
  app$setInputs(resetbnt = "click")
  expect_equal(app$getValue("slider_inp_example"), 3)
})

-> takeScreenshot() from the shinytest package aims to control the visuals of the app, whereas recordTest() can be used to test both UI and server by recording the use of the app and testing against the recording.

-> Finally, have a look at the shinyloadtest package to better control the performance of your app.

6. Pluck up your courage and go ahead with the great spring cleaning action: afterwards it will be much nicer and easier to work.
