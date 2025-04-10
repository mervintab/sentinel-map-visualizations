# Microsoft Sentinel Map Visualizations

This repository contains custom map visualizations for use in **Microsoft Sentinel Workbooks**. Each JSON file is configured to display data on a map using **Kusto Query Language (KQL)** with various settings for geo-coordinates, aggregation, and heatmap styles.

---

## 📁 Contents

- [`signinlogs_map.json`](maps/signinlogs_map.json) — Successful login heatmap (Entra ID (Azure) Authentication Success)
- [`failedlogins_map.json`](maps/failedlogins_map.json) — Failed login heatmap (Entra ID (Azure) Authentication Failures)
- [`maliciousflows_map.json`](maps/maliciousflows_map.json) — Malicious network flow visualization (Malicious Traffic Entering the Network)
- [`resourcecreation_map.json`](maps/resourcecreation_map.json) — Resource creation activity by IP (Azure Resource Creation)
- [`failedlogon_remoteip_map.json`](maps/failedlogon_remoteip_map.json) — Remote IP failed logon attempts (VM Authentication Failures)

---

### 🔹 [`signinlogs_map.json`](maps/signinlogs_map.json)

**Purpose**:  
Displays successful login activity (ResultType = 0) from Azure Active Directory `SigninLogs`, showing a heatmap based on login counts by location.

**Features**:
- Heatmap using latitude and longitude from `LocationDetails`
- Size and color based on number of logins (`LoginCount`)
- Labels each marker with user and location
- Aggregated by `Identity`, `City`, and `Country`

**KQL Summary**:
```kql
SigninLogs
| where ResultType == 0
| summarize LoginCount = count() by Identity,
    Latitude = tostring(LocationDetails["geoCoordinates"]["latitude"]),
    Longitude = tostring(LocationDetails["geoCoordinates"]["longitude"]),
    City = tostring(LocationDetails["city"]),
    Country = tostring(LocationDetails["countryOrRegion"])
| project Identity, Latitude, Longitude, City, Country, LoginCount,
    friendly_label = strcat(Identity, " - ", City, ", ", Country)
```

---

### 🔹 [`failedlogins_map.json`](maps/failedlogins_map.json)

**Purpose**:  
This map visualization displays **failed sign-in attempts** (i.e., `ResultType != 0`) captured from Azure AD `SigninLogs` over the past 30 days. It helps identify **geographic patterns of unsuccessful login activity**, which may indicate brute-force attacks, password spray attempts, or unauthorized access efforts.

**Features**:
- Heatmap visualization using `Latitude` and `Longitude`
- Colored and sized by `LoginCount`
- Green to red palette based on login failure volume
- Location-based aggregation

**KQL Summary**:
```kql
SigninLogs
| where ResultType != 0
| summarize LoginCount = count() by Identity,
    Latitude = tostring(LocationDetails["geoCoordinates"]["latitude"]),
    Longitude = tostring(LocationDetails["geoCoordinates"]["longitude"]),
    City = tostring(LocationDetails["city"]),
    Country = tostring(LocationDetails["countryOrRegion"])
| project Identity, Latitude, Longitude, City, Country, LoginCount,
    friendly_label = strcat(Identity, " - ", City, ", ", Country)
```

---

### 🔹 [`maliciousflows_map.json`](maps/maliciousflows_map.json)

**Purpose**:  
This map visualizes **malicious network flows** detected by `AzureNetworkAnalytics_CL`, enriched with IP geolocation data from a custom **GeoIP watchlist** (`geoip`). It provides a geographic view of potential attacker infrastructure or anomalous traffic sources over a 30-day window.

**Features**:
- Uses `ipv4_lookup()` to match malicious source IPs with geolocation data
- Displays heatmap by **city** using `latitude` and `longitude`
- Aggregates color and size based on city-level traffic volume
- Labels each point with a **friendly location** (`City (Country)`)
- Highlights high-density areas using a green-to-red heatmap

**KQL Summary**:
```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
let MaliciousFlows = AzureNetworkAnalytics_CL 
| where FlowType_s == "MaliciousFlow"
| order by TimeGenerated desc
| project TimeGenerated, FlowType = FlowType_s, IpAddress = SrcIP_s,
         DestinationIpAddress = DestIP_s, DestinationPort = DestPort_d,
         Protocol = L7Protocol_s, NSGRuleMatched = NSGRules_s;
MaliciousFlows
| evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network)
| project TimeGenerated, FlowType, IpAddress, DestinationIpAddress,
         DestinationPort, Protocol, NSGRuleMatched, latitude, longitude,
         city = cityname, country = countryname,
         friendly_location = strcat(cityname, " (", countryname, ")")
```

---

### 🔹 [`resourcecreation_map.json`](maps/resourcecreation_map.json)

**Purpose**:  
This map visualizes **successful Azure resource creation events** by public IP address, enriched with location data using a custom `geoip` watchlist. It is useful for identifying geographic patterns in deployment activity and spotting suspicious or unexpected regions of origin.

**Features**:
- Filters out system accounts and focuses on named users or service principals
- Includes only successful `WRITE` operations (i.e., actual resource creation)
- Performs IP enrichment using a custom `geoip` watchlist
- Labels map points with `CallerPrefix - City, Country`
- Uses a heatmap to reflect the number of resources created from each location

**KQL Summary**:
```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
let AzureActivityRecords = AzureActivity
| where not(Caller matches regex @guid_regex)
| where CallerIpAddress matches regex @"\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b"
| where OperationNameValue endswith "WRITE"
  and (ActivityStatusValue == "Success" or ActivityStatusValue == "Succeeded")
| summarize ResouceCreationCount = count() by Caller, CallerIpAddress;
AzureActivityRecords
| evaluate ipv4_lookup(GeoIPDB_FULL, CallerIpAddress, network)
| project Caller,
         CallerPrefix = split(Caller, "@")[0],
         CallerIpAddress,
         ResouceCreationCount,
         Country = countryname,
         Latitude = latitude,
         Longitude = longitude,
         friendly_label = strcat(split(Caller, "@")[0], " - ", cityname, ", ", countryname)
```

---

### 🔹 [`failedlogon_remoteip_map.json`](maps/failedlogon_remoteip_map.json)

**Purpose**:  
This map visualizes **failed logon attempts** from remote IP addresses, using data from `DeviceLogonEvents`. It enriches IPs with location data from the `geoip` watchlist and displays a heatmap based on the number of failed attempts over a 30-day period.

**Features**:
- Filters for logon failures (`ActionType == "LogonFailed"`)
- Enriches `RemoteIP` with city and country information using `ipv4_lookup()`
- Summarizes failed logon counts by location
- Uses heatmap coloring and size scaling based on number of attempts
- Labels points with the format: `City (Country)`

**KQL Summary**:
```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
DeviceLogonEvents
| where ActionType == "LogonFailed"
| order by TimeGenerated desc
| evaluate ipv4_lookup(GeoIPDB_FULL, RemoteIP, network)
| summarize LoginAttempts = count() by 
    RemoteIP, 
    City = cityname, 
    Country = countryname, 
    friendly_location = strcat(cityname, " (", countryname, ")"), 
    Latitude = latitude, 
    Longitude = longitude;
```

---

## 🚀 How to Use

1. Open your Microsoft Sentinel Workbook.
2. Click **"Add query"** and choose **"Map"** as the visualization type.
3. Paste the KQL query from the `.json` file.
4. Apply the map settings to match those defined in the file (or import the JSON if supported).
5. Customize as needed for your environment.

---

## 🔐 Author

**Mervin Tabernero**  
Cybersecurity Analyst in training | Microsoft Sentinel Enthusiast

---

## 📝 License

This project is licensed under the MIT License. Feel free to use and adapt!
