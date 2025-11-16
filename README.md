# 🏥 Healthcare Analytics Dashboard (R Shiny)

This project is an interactive **R Shiny dashboard** built to analyze a real healthcare dataset related to **stroke prediction**.  
It visualizes data on age, BMI, glucose levels, smoking status, and other health factors.

---

## 🚀 Live Demo
**Hosted App:** [https://charan37.shinyapps.io/healthcaredashboard/](https://charan37.shinyapps.io/healthcaredashboard/)

---

## 📊 Features
- Interactive filters for gender, age range, and smoking status  
- Real-time plots using **Plotly**  
- Key statistics such as total patients, average glucose, and BMI  
- Dynamic data table with search and pagination  

---

## ⚙️ How to Run Locally
1. Clone this repo or download the ZIP file.
2. Open RStudio and set your working directory to this folder.
3. Install required libraries:
   ```r
   install.packages(c("shiny", "shinythemes", "plotly", "dplyr", "DT", "readr"))
