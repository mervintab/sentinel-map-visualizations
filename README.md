# Microsoft Sentinel Map Visualizations

This repository contains custom map visualizations for use in **Microsoft Sentinel Workbooks**. Each JSON file is configured to display data on a map using **Kusto Query Language (KQL)** with various settings for geo-coordinates, aggregation, and heatmap styles.

---

## 📁 Contents

### 🔹 `signinlogs_map.json`

- [`signinlogs_map.json`](maps/signinlogs_map.json) — Successful login heatmap




**Purpose**:  
Displays successful login activity (ResultType = 0) from Azure Active Directory `SigninLogs`, showing a heatmap based on login counts by location.


### 🔹 `failedlogins_map.json`
- [`failedlogins_map.json`](maps/failedlogins_map.json) — Failed login heatmap
**Purpose**:  
This map visualization displays **failed sign-in attempts** (i.e., `ResultType != 0`) captured from Azure AD `SigninLogs` over the past 30 days. It helps identify **geographic patterns of unsuccessful login activity**, which may indicate brute-force attacks, password spray attempts, or unauthorized access efforts.

**Features**:
- Heatmap visualization using `Latitude` and `Longitude`
- Colored and sized by `LoginCount`
- Green to red palette based on login failure volume
- Location-based aggregation

### 🔹 [`maliciousflows_map.json`](maps/maliciousflows_map.json)
- [`maliciousflows_map.json`](maps/maliciousflows_map.json) — Malicious network flow visualization
**Purpose**:  
This map visualizes **malicious network flows** detected by `AzureNetworkAnalytics_CL`, enriched with IP geolocation data from a custom **GeoIP watchlist** (`geoip`). It provides a geographic view of potential attacker infrastructure or anomalous traffic sources over a 30-day window.

**Features**:
- Uses `ipv4_lookup()` to match malicious source IPs with geolocation data
- Displays heatmap by **city** using `latitude` and `longitude`
- Aggregates color and size based on city-level traffic volume
- Labels each point with a **friendly location** (`City (Country)`)
- Highlights high-density areas using a green-to-red heatmap

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
