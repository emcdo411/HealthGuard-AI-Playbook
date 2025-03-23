# HealthPulse Analytics

## Overview
*HealthPulse Analytics* is a Python and RStudio-powered project that emulates the workflow of a Health Data Scientist in healthcare IT. As a U.S. Army veteran passionate about leveraging technology to improve lives, I built this to analyze synthetic datasets—patient records, IT support tickets, and compliance logs—mirroring real-world challenges like HIPAA compliance, IT productivity, and patient care support. This repository demonstrates my expertise in data generation, advanced visualization, and healthcare analytics.

---

## Table of Contents
- [Datasets](#datasets)
- [Code](#code)
  - [Python Scripts](#python-scripts)
  - [R Scripts](#r-scripts)
- [Setup Instructions](#setup-instructions)
- [Why It Matters](#why-it-matters)
- [Conclusion](#conclusion)
- [License](#license)

---

## Datasets
This project includes three `.csv` files, each generated with Python and visualized in RStudio:

1. **`it_tickets.csv`**:
   - **Columns**: `Ticket_ID`, `User_Role`, `Department`, `Issue_Description`, `PHI_Detected`, `Priority`, `Submission_Date`, `Resolution_Date`, `Status`
   - **Rows**: 300
   - **Description**: Simulates IT support tickets in a healthcare setting, tracking issues like EHR login failures.

2. **`compliance_logs.csv`**:
   - **Columns**: `Log_ID`, `Ticket_ID`, `PHI_Redaction_Rate`, `Compliance_Score`, `Region`, `Latitude`, `Longitude`, `Log_Date`
   - **Rows**: 300
   - **Description**: Logs HIPAA compliance metrics tied to IT tickets, with geographic data for regional analysis.

3. **`patient_records.csv`**:
   - **Columns**: `Patient_ID`, `Name`, `SSN`, `Address`, `Physician`, `Department`, `Last_Visit_Date`
   - **Rows**: 200
   - **Description**: Synthetic patient records for analyzing departmental workloads and visit trends.

---

## Code

### Python Scripts

#### 1. `generate_it_tickets.py`
```python
import pandas as pd
import numpy as np
from faker import Faker
import random

fake = Faker()
np.random.seed(42)

def generate_it_tickets(n=300):
    roles = ['Nurse', 'Physician', 'Admin']
    departments = ['Cardiology', 'Neurology', 'Oncology', 'Pediatrics', 'Orthopedics']
    issues = ['EHR Login Failure', 'System Crash', 'Data Access Issue', 'Printer Failure']
    data = {
        'Ticket_ID': [f"INC{str(i).zfill(4)}" for i in range(1, n+1)],
        'User_Role': random.choices(roles, k=n),
        'Department': random.choices(departments, k=n),
        'Issue_Description': [f"{random.choice(issues)} for {fake.name()}, Patient ID: PT{str(random.randint(1, 200)).zfill(3)}" for _ in range(n)],
        'PHI_Detected': random.choices([True, False], weights=[70, 30], k=n),
        'Priority': random.choices(['High', 'Medium', 'Low'], weights=[30, 50, 20], k=n),
        'Submission_Date': [fake.date_time_between(start_date='-1y', end_date='today').strftime('%Y-%m-%d %H:%M:%S') for _ in range(n)],
        'Resolution_Date': [fake.date_time_between_dates(datetime_start=fake.date_time_between(start_date='-1y', end_date='today'), end_date='today').strftime('%Y-%m-%d %H:%M:%S') if random.random() > 0.1 else '' for _ in range(n)],
        'Status': []  # To be filled based on Resolution_Date
    }
    data['Status'] = [random.choice(['Open', 'Resolved']) if res_date else 'Open' for res_date in data['Resolution_Date']]
    return pd.DataFrame(data)

tickets_df = generate_it_tickets(300)
tickets_df.to_csv('it_tickets.csv', index=False)
print("Generated 'it_tickets.csv'")
```

#### 2. `generate_compliance_logs.py`
```python
import pandas as pd
import numpy as np
from faker import Faker
import random

fake = Faker()
np.random.seed(42)

us_cities = {
    'New York City, NY': (40.7128, -74.0060), 'Chicago, IL': (41.8781, -87.6298),
    'Houston, TX': (29.7604, -95.3698), 'Los Angeles, CA': (34.0522, -118.2437),
    'Phoenix, AZ': (33.4484, -112.0740), 'Seattle, WA': (47.6062, -122.3321),
    'Miami, FL': (25.7617, -80.1918), 'Denver, CO': (39.7392, -104.9903),
    'Atlanta, GA': (33.7490, -84.3880), 'Boston, MA': (42.3601, -71.0589)
}

def generate_compliance_logs(n=300):
    cities = list(us_cities.keys())
    ticket_ids = [f"INC{str(i).zfill(4)}" for i in range(1, n+1)]
    data = {
        'Log_ID': [f"LOG{str(i).zfill(4)}" for i in range(1, n+1)],
        'Ticket_ID': ticket_ids,
        'PHI_Redaction_Rate': [1.0 if random.random() > 0.1 else random.uniform(0.8, 0.99) for _ in range(n)],
        'Compliance_Score': [random.uniform(0.9, 1.0) if r == 1.0 else random.uniform(0.7, 0.9) for r in data['PHI_Redaction_Rate']],
        'Region': random.choices(cities, k=n),
        'Latitude': [us_cities[city][0] for city in random.choices(cities, k=n)],
        'Longitude': [us_cities[city][1] for city in random.choices(cities, k=n)],
        'Log_Date': [fake.date_between(start_date='-1y', end_date='today').strftime('%Y-%m-%d') for _ in range(n)]
    }
    return pd.DataFrame(data)

compliance_df = generate_compliance_logs(300)
compliance_df.to_csv('compliance_logs.csv', index=False)
print("Generated 'compliance_logs.csv'")
```

#### 3. `generate_patient_records.py`
```python
import pandas as pd
import numpy as np
from faker import Faker
import random

fake = Faker()
np.random.seed(42)

def generate_patient_records(n=200):
    departments = ['Cardiology', 'Neurology', 'Oncology', 'Pediatrics', 'Orthopedics']
    data = {
        'Patient_ID': [f"PT{str(i).zfill(3)}" for i in range(1, n+1)],
        'Name': [fake.name() for _ in range(n)],
        'SSN': [fake.ssn() for _ in range(n)],
        'Address': [fake.address().replace('\n', ', ') for _ in range(n)],
        'Physician': [fake.name() for _ in range(n)],
        'Department': random.choices(departments, k=n),
        'Last_Visit_Date': [fake.date_between(start_date='-1y', end_date='today').strftime('%Y-%m-%d') for _ in range(n)]
    }
    return pd.DataFrame(data)

patient_df = generate_patient_records(200)
patient_df.to_csv('patient_records.csv', index=False)
print("Generated 'patient_records.csv'")
```

### R Scripts

#### 1. `ticket_resolution_time.R`
```R
if (!require(ggplot2)) install.packages("ggplot2"); library(ggplot2)
if (!require(dplyr)) install.packages("dplyr"); library(dplyr)
if (!require(plotly)) install.packages("plotly"); library(plotly)

setwd("C:/Users/Veteran")
csv_file <- "it_tickets.csv"
if (!file.exists(csv_file)) stop("Error: it_tickets.csv not found")

tickets <- read.csv("it_tickets.csv")
tickets$Submission_Date <- as.POSIXct(tickets$Submission_Date, format="%Y-%m-%d %H:%M:%S", tz="UTC")
tickets$Resolution_Date <- as.POSIXct(tickets$Resolution_Date, format="%Y-%m-%d %H:%M:%S", tz="UTC")
tickets <- tickets %>% 
  mutate(Resolution_Time_Hours = ifelse(Status == "Resolved" & !is.na(Resolution_Date), 
                                        as.numeric(difftime(Resolution_Date, Submission_Date, units="hours")), 
                                        NA)) %>%
  filter(!is.na(Resolution_Time_Hours))

ticket_boxplot <- ggplot(tickets, aes(x = Department, y = Resolution_Time_Hours, fill = Priority)) +
  geom_boxplot() +
  theme_minimal() +
  labs(title = "IT Ticket Resolution Time by Department (Boxplot)", 
       x = "Department", 
       y = "Resolution Time (Hours)") +
  theme(legend.position = "right", 
        plot.title = element_text(hjust = 0.5, size = 16, face = "bold"))

print("Displaying static boxplot:")
print(ticket_boxplot)

ticket_scatter <- ggplot(tickets, aes(x = Department, y = Resolution_Time_Hours, color = Priority, 
                                      text = paste("Ticket ID:", Ticket_ID))) +
  geom_jitter(width = 0.2, alpha = 0.7, size = 2) +
  theme_minimal() +
  labs(title = "IT Ticket Resolution Time by Department (Scatter)", 
       x = "Department", 
       y = "Resolution Time (Hours)", 
       color = "Priority") +
  theme(legend.position = "right", 
        plot.title = element_text(hjust = 0.5, size = 16, face = "bold"))

print("Displaying static scatter plot:")
print(ticket_scatter)

ticket_plotly <- ggplotly(ticket_scatter, tooltip = c("x", "y", "color", "text"))
print("Displaying interactive scatter plot:")
print(ticket_plotly)

ggsave("ticket_resolution_time_boxplot.png", ticket_boxplot, width = 10, height = 6, dpi = 300)
ggsave("ticket_resolution_time_scatter.png", ticket_scatter, width = 10, height = 6, dpi = 300)
htmlwidgets::saveWidget(ticket_plotly, "ticket_resolution_time.html")
cat("Saved as 'ticket_resolution_time_boxplot.png', 'ticket_resolution_time_scatter.png', and 'ticket_resolution_time.html'\n")
```

#### 2. `compliance_map.R`
```R
if (!require(ggplot2)) install.packages("ggplot2"); library(ggplot2)
if (!require(ggmap)) install.packages("ggmap"); library(ggmap)
if (!require(plotly)) install.packages("plotly"); library(plotly)
if (!require(viridis)) install.packages("viridis"); library(viridis)

register_stadiamaps("9c644007-0572-4892-915a-8da356fe40ae")
setwd("C:/Users/Veteran")
csv_file <- "compliance_logs.csv"
if (!file.exists(csv_file)) stop("Error: compliance_logs.csv not found")

compliance <- read.csv("compliance_logs.csv")
us_bbox <- c(left = -125, bottom = 24, right = -66, top = 50)
us_terrain <- get_stadiamap(bbox = us_bbox, maptype = "stamen_terrain", zoom = 5)
if (is.null(us_terrain)) stop("Error: Failed to load Stadia Maps terrain data.")

compliance_base <- ggmap(us_terrain) +
  geom_point(data = compliance, aes(x = Longitude, y = Latitude, color = Compliance_Score, size = PHI_Redaction_Rate), 
             alpha = 0.7, shape = 16) +
  scale_color_viridis_c(option = "viridis", direction = -1) +
  scale_size_continuous(range = c(2, 8)) +
  labs(title = "HIPAA Compliance Across U.S. Regions",
       subtitle = "Point size reflects PHI Redaction Rate",
       x = "Longitude", y = "Latitude", 
       color = "Compliance Score", size = "PHI Redaction Rate") +
  theme_minimal() +
  theme(legend.position = "right",
        plot.title = element_text(hjust = 0.5, size = 16, face = "bold"),
        plot.subtitle = element_text(hjust = 0.5, size = 12))

print("Displaying static map:")
print(compliance_base)

compliance_plotly <- ggplotly(compliance_base, 
                             tooltip = c("x", "y", "color", "size", "text"),
                             text = ~paste("Region:", Region, "<br>Ticket ID:", Ticket_ID, "<br>Log Date:", Log_Date))
print("Displaying interactive map:")
print(compliance_plotly)

ggsave("compliance_map.png", compliance_base, width = 12, height = 8, dpi = 300)
htmlwidgets::saveWidget(compliance_plotly, "compliance_map.html")
cat("Saved as 'compliance_map.png' and 'compliance_map.html'\n")
```

#### 3. `patient_visits.R`
```R
if (!require(ggplot2)) install.packages("ggplot2"); library(ggplot2)
if (!require(dplyr)) install.packages("dplyr"); library(dplyr)
if (!require(plotly)) install.packages("plotly"); library(plotly)

setwd("C:/Users/Veteran")
csv_file <- "patient_records.csv"
if (!file.exists(csv_file)) stop("Error: patient_records.csv not found")

patients <- read.csv("patient_records.csv")
patients$Last_Visit_Date <- as.Date(patients$Last_Visit_Date, format="%Y-%m-%d")

dept_counts <- patients %>% 
  group_by(Department) %>% 
  summarise(Patient_Count = n())

bar_plot <- ggplot(dept_counts, aes(x = Department, y = Patient_Count, fill = Department)) +
  geom_bar(stat = "identity") +
  theme_minimal() +
  labs(title = "Patient Distribution by Department", 
       x = "Department", 
       y = "Number of Patients") +
  theme(legend.position = "none", 
        plot.title = element_text(hjust = 0.5, size = 16, face = "bold"),
        axis.text.x = element_text(angle = 45, hjust = 1))

print("Displaying static bar plot:")
print(bar_plot)

visit_trend <- patients %>% 
  group_by(Last_Visit_Date) %>% 
  summarise(Visit_Count = n())

time_plot <- ggplot(visit_trend, aes(x = Last_Visit_Date, y = Visit_Count)) +
  geom_line(color = "blue") +
  geom_point(color = "blue", size = 2) +
  theme_minimal() +
  labs(title = "Patient Visits Over Time", 
       x = "Date", 
       y = "Number of Visits") +
  scale_x_date(date_breaks = "1 month", date_labels = "%b %Y") +
  theme(plot.title = element_text(hjust = 0.5, size = 16, face = "bold"),
        axis.text.x = element_text(angle = 45, hjust = 1))

print("Displaying static time series plot:")
print(time_plot)

time_plotly <- ggplotly(time_plot, tooltip = c("x", "y"))
print("Displaying interactive time series plot:")
print(time_plotly)

ggsave("patient_dept_bar.png", bar_plot, width = 8, height = 6, dpi = 300)
ggsave("patient_visit_trend.png", time_plot, width = 10, height = 6, dpi = 300)
htmlwidgets::saveWidget(time_plotly, "patient_visit_trend.html")
cat("Saved as 'patient_dept_bar.png', 'patient_visit_trend.png', and 'patient_visit_trend.html'\n")
```

---

## Setup Instructions

### Prerequisites
- **Python**: Install required packages:
  ```bash
  pip install pandas numpy faker
  ```
- **R**: Install RStudio and required packages:
  ```R
  install.packages(c("ggplot2", "dplyr", "plotly", "ggmap", "viridis"))
  ```
- **Stadia Maps API Key**: Use `9c644007-0572-4892-915a-8da356fe40ae` (or get your own at [Stadia Maps](https://stadiamaps.com/)).

### Steps
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/yourusername/healthpulse-analytics.git
   cd healthpulse-analytics
   ```
2. **Generate Datasets**:
   - Run each Python script:
     ```bash
     python generate_it_tickets.py
     python generate_compliance_logs.py
     python generate_patient_records.py
     ```
   - Move `.csv` files to `C:/Users/Veteran` (or adjust `setwd()` in R scripts).
3. **Visualize Data**:
   - Open RStudio, load each `.R` script, and run (`Ctrl+Alt+R`).
   - Outputs saved as `.png` (static) and `.html` (interactive) in `C:/Users/Veteran`.

---

## Why It Matters
This project bridges healthcare IT and data science, offering actionable insights for real-world challenges:
- **IT Productivity**: The ticket resolution time visualizations (`it_tickets.csv`) reveal how quickly IT issues are fixed by department and priority. Faster resolutions mean less downtime for clinicians, directly improving patient care delivery.
- **HIPAA Compliance**: The compliance map (`compliance_logs.csv`) highlights regional trends in PHI redaction and compliance scores. For a Health Data Scientist, this identifies areas needing stricter protocols, reducing legal and ethical risks.
- **Patient Care Insights**: The patient visit analyses (`patient_records.csv`) show departmental workloads and visit trends over time. This helps healthcare IT teams anticipate resource needs—e.g., more support for Cardiology during peak seasons—enhancing operational efficiency.
- **Professional Impact**: As a portfolio piece, it showcases my ability to generate, analyze, and visualize healthcare data, aligning with Health Data Scientist roles that demand both technical prowess and domain knowledge.

---

## Conclusion
*HealthPulse Analytics* is a testament to the power of Python and RStudio in healthcare analytics. From synthetic data generation to interactive visualizations, it emulates a Health Data Scientist’s toolkit with flair. Whether you’re exploring IT ticket resolution, mapping compliance, or tracking patient visits, this project invites you to dive in, adapt the code, and uncover insights. Clone it, run it, and let’s make healthcare IT smarter together!

---

## License
This project is licensed under the MIT License—see the `LICENSE` file for details.

---

### Notes for GitHub
- **Repository Structure**: Add all `.py`, `.R`, and `.csv` files to your repo. Include sample outputs (e.g., `.png` files) in an `/outputs` folder for visual appeal.
- **Personalize**: Replace `https://github.com/yourusername/healthpulse-analytics.git` with your actual repo URL. Let me know your GitHub username if you want me to update it!
- **Stadia Maps API**: If sharing publicly, consider removing the API key from `compliance_map.R` and instructing users to get their own, or keep it private if this is personal.

This README is ready to shine on GitHub—comprehensive, engaging, and a perfect showcase of your work. What’s your GitHub username so I can tailor the clone URL? Any final tweaks you’d like before pushing it live?
- **Compliance:** Embeds HL7 and HIPAA safeguards, reducing risk of violations.
- **Patient Care:** Ensures clinicians (e.g., Dr. Smith) get quick IT support, improving care delivery.
- **Professional Impact:** Showcases my expertise in healthcare IT, AI integration, and compliance governance—key for Business Analyst roles.

---

## Future Enhancements

- **Expand MuleSoft Use Cases:** Add lab result integration (ORU messages) and appointment scheduling sync.
- **Enhance Dashboard:** Add widgets for “Top Ticket Categories” and “User Activity Trends.”
- **Deploy to Users:** Roll out to a small healthcare IT team, gather feedback, and iterate.

---

## About Me

I’m a US Army veteran and Business Systems Analyst with over 10 years in healthcare IT, passionate about using technology to improve lives. Connect with me on LinkedIn: [Insert LinkedIn URL].

---

## License

This project is licensed under the MIT License—see the [LICENSE](LICENSE) file for details.
