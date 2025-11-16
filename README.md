# --- Healthcare Analytics Dashboard (Professional Version) ---

library(shiny)
library(shinydashboard)
library(plotly)
library(dplyr)
library(DT)
library(readr)

# --- Load Dataset ---
hd <- read_csv("healthcare-dataset-stroke-data.csv")
hd$bmi <- as.numeric(hd$bmi)
hd$avg_glucose_level <- as.numeric(hd$avg_glucose_level)

# --- UI ---
ui <- dashboardPage(
  dashboardHeader(title = "🏥 Healthcare Analytics Dashboard"),
  
  dashboardSidebar(
    sidebarMenu(
      menuItem("Overview", tabName = "overview", icon = icon("home")),
      menuItem("Analytics", tabName = "analytics", icon = icon("chart-line")),
      menuItem("Patient Data", tabName = "data", icon = icon("table")),
      menuItem("About Project", tabName = "about", icon = icon("info-circle"))
    )
  ),
  
  dashboardBody(
    tabItems(
      # --- Overview Tab ---
      tabItem(tabName = "overview",
              fluidRow(
                box(width = 12, solidHeader = TRUE, status = "primary",
                    h3("🏥 Healthcare Data Summary", align = "center"),
                    p("This dashboard analyzes stroke-related health data, exploring patterns in glucose levels, BMI, age, and lifestyle factors like smoking and gender.",
                      align = "center")
                )
              ),
              fluidRow(
                valueBoxOutput("totalPatients", width = 4),
                valueBoxOutput("avgGlucose", width = 4),
                valueBoxOutput("avgBMI", width = 4)
              ),
              fluidRow(
                box(title = "Age Distribution by Stroke Outcome", width = 12, status = "info",
                    plotlyOutput("ageStrokePlot", height = "350px"))
              )
      ),
      
      # --- Analytics Tab ---
      tabItem(tabName = "analytics",
              fluidRow(
                box(title = "Filters", width = 3, solidHeader = TRUE, status = "primary",
                    selectInput("gender", "Select Gender:", choices = c("All", unique(hd$gender))),
                    sliderInput("ageRange", "Select Age Range:",
                                min = min(hd$age, na.rm = TRUE),
                                max = max(hd$age, na.rm = TRUE),
                                value = c(min(hd$age, na.rm = TRUE), max(hd$age, na.rm = TRUE))),
                    selectInput("smoke", "Smoking Status:",
                                choices = c("All", unique(hd$smoking_status)))
                ),
                box(title = "Age vs Glucose Level", width = 9, status = "warning",
                    plotlyOutput("glucosePlot", height = "300px"))
              ),
              fluidRow(
                box(title = "Age vs BMI", width = 6, status = "success",
                    plotlyOutput("bmiPlot", height = "300px")),
                box(title = "Gender vs Stroke Outcome", width = 6, status = "danger",
                    plotlyOutput("genderStrokePlot", height = "300px"))
              )
      ),
      
      # --- Data Tab ---
      tabItem(tabName = "data",
              box(width = 12, title = "Patient Data (Filtered)", status = "primary", solidHeader = TRUE,
                  DTOutput("table"))
      ),
      
      # --- About Tab ---
      tabItem(tabName = "about",
              box(width = 12, status = "info", solidHeader = TRUE,
                  h3("About This Dashboard"),
                  p("Developed using R Shiny and Plotly, this dashboard visualizes stroke-related healthcare data. 
                    It helps explore relationships between patient attributes and stroke occurrence, 
                    supporting data-driven decision-making in healthcare."),
                  hr(),
                  h4("📊 Key Technologies Used:"),
                  tags$ul(
                    tags$li("R Shiny for interactive UI"),
                    tags$li("Plotly for visual analytics"),
                    tags$li("dplyr for data manipulation"),
                    tags$li("DT for interactive data tables")
                  ),
                  h4("👨‍💻 Created by: Charan Pasunuri"),
                  h5("Data Science & Analytics Enthusiast")
              )
      )
    )
  )
)

# --- SERVER ---
server <- function(input, output) {
  filteredData <- reactive({
    data <- hd
    if (input$gender != "All") data <- data %>% filter(gender == input$gender)
    if (input$smoke != "All") data <- data %>% filter(smoking_status == input$smoke)
    data <- data %>% filter(age >= input$ageRange[1], age <= input$ageRange[2])
    return(data)
  })
  
  # Value boxes
  output$totalPatients <- renderValueBox({
    valueBox(nrow(filteredData()), "Total Patients", icon = icon("users"), color = "aqua")
  })
  
  output$avgGlucose <- renderValueBox({
    valueBox(round(mean(filteredData()$avg_glucose_level, na.rm = TRUE), 2),
             "Avg Glucose Level", icon = icon("tint"), color = "green")
  })
  
  output$avgBMI <- renderValueBox({
    valueBox(round(mean(filteredData()$bmi, na.rm = TRUE), 2),
             "Avg BMI", icon = icon("weight"), color = "yellow")
  })
  
  # Charts
  output$ageStrokePlot <- renderPlotly({
    plot_ly(filteredData(), x = ~age, color = ~factor(stroke),
            type = "histogram") %>%
      layout(title = "Age Distribution by Stroke Outcome",
             xaxis = list(title = "Age"), yaxis = list(title = "Count"))
  })
  
  output$glucosePlot <- renderPlotly({
    plot_ly(filteredData(), x = ~age, y = ~avg_glucose_level, color = ~factor(stroke),
            type = "scatter", mode = "markers") %>%
      layout(title = "Age vs Glucose Level",
             xaxis = list(title = "Age"), yaxis = list(title = "Avg Glucose Level"))
  })
  
  output$bmiPlot <- renderPlotly({
    plot_ly(filteredData(), x = ~age, y = ~bmi, color = ~factor(stroke),
            type = "scatter", mode = "markers") %>%
      layout(title = "Age vs BMI",
             xaxis = list(title = "Age"), yaxis = list(title = "BMI"))
  })
  
  output$genderStrokePlot <- renderPlotly({
    gender_summary <- filteredData() %>%
      group_by(gender, stroke) %>%
      summarise(count = n(), .groups = "drop")
    plot_ly(gender_summary, x = ~gender, y = ~count, color = ~factor(stroke),
            type = "bar", barmode = "group") %>%
      layout(title = "Gender vs Stroke Outcome",
             xaxis = list(title = "Gender"), yaxis = list(title = "Count"))
  })
  
  output$table <- renderDT({
    datatable(filteredData(), options = list(pageLength = 10, scrollX = TRUE))
  })
}

# --- Run App ---
shinyApp(ui = ui, server = server)
