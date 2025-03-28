# Microsoft Sentinel Map Visualizations

This repository contains custom map visualizations for use in **Microsoft Sentinel Workbooks**. Each JSON file is configured to display data on a map using **Kusto Query Language (KQL)** with various settings for geo-coordinates, aggregation, and heatmap styles.

---

## 📁 Contents

### 🔹 `signinlogs_map.json`

**Purpose**:  
Displays successful login activity (ResultType = 0) from Azure Active Directory `SigninLogs`, showing a heatmap based on login counts by location.

---

## 🚀 How to Use

1. Open your Microsoft Sentinel Workbook.
2. Click **"Add query"** and choose **"Map"** as the visualization type.
3. Paste the KQL query from the `.json` file.
4. Apply the map settings to match those defined in the file (or import the JSON if supported).
5. Customize as needed for your environment.

---

## 📌 Notes

- These visualizations are designed for **custom SOC dashboards** and **geo-visual analysis**.
- This is a growing collection — feel free to fork and contribute!

---

## 🔐 Author

**Mervin Tabernero**  
Cybersecurity Analyst in training | Microsoft Sentinel Enthusiast

---

## 📝 License

This project is licensed under the MIT License. Feel free to use and adapt!
