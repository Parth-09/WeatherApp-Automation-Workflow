# WeatherApp-Automation-Workflow
# 🌤️ Daily Weather Automation with n8n

An automated weather monitoring system that fetches daily weather data for multiple cities, generates intelligent alerts, stores historical data, and delivers beautiful email digests.

## ✨ Features

- 🌍 Multi-city monitoring (London, New York, Tokyo, Sydney, Paris)
- ⚠️ Smart weather alerts (precipitation, heat, frost, wind)
- 📊 Historical data storage in Supabase
- 📧 Beautiful HTML email digest
- 🔄 Automated daily execution

## 🛠️ Setup Instructions

### 1. OpenWeatherMap API

1. Sign up at [OpenWeatherMap](https://openweathermap.org/api)
2. Get your API key from the dashboard
3. Add the key to the "Loop Through 5 Cities" node in the workflow

### 2. Supabase Database

**Create Project:**
- Sign up at [Supabase](https://supabase.com)
- Create a new project

**Run this SQL in Supabase SQL Editor:**
```sql
CREATE TABLE weather_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  city TEXT NOT NULL,
  temperature FLOAT NOT NULL,
  temperature_unit TEXT DEFAULT 'F',
  condition TEXT,
  humidity INT,
  wind_speed FLOAT,
  alert_type TEXT,
  raw_response JSONB
);

CREATE INDEX idx_weather_logs_run_at ON weather_logs(run_at DESC);
CREATE INDEX idx_weather_logs_city ON weather_logs(city);
```

**Get Credentials:**
- Go to Settings > API
- Copy your Project URL and anon/public key
- Add these to the "Append to Supabase" node

### 3. Email Configuration

**Using Gmail:**
1. Enable 2-Factor Authentication
2. Generate an App Password (Google Account > Security > App Passwords)
3. Configure in "Send Email" node:
   - SMTP: `smtp.gmail.com`
   - Port: `465` (SSL) or `587` (TLS)
   - Username: Your Gmail address
   - Password: App Password

**Using Other SMTP:**
- Configure your SMTP provider details in the "Send Email" node

### 4. Import Workflow

1. Download `weather_automation.json` from this repository
2. Open n8n (cloud or self-hosted)
3. Click "Import from File"
4. Select the downloaded file
5. Update credentials in these nodes:
   - Loop Through 5 Cities (OpenWeather API key)
   - Append to Supabase (Supabase URL and key)
   - Send Email (SMTP credentials)

### 5. Configure Schedule

- Default: Runs daily at 8:00 AM
- To change: Edit the "Schedule Trigger" node
- For testing: Set to run every 5 minutes (`*/5 * * * *`)

## 🎯 Workflow Overview
```
Schedule → Config → Fetch Weather → Extract Data → Save to DB → Aggregate → Send Email
```

1. **Schedule Trigger** - Runs daily at set time
2. **Loop Through 5 Cities** - Defines cities and configuration
3. **Fetch Weather** - Calls OpenWeatherMap API (5x)
4. **Extract City Info** - Parses and analyzes data (5x)
5. **Append to Supabase** - Stores in database (5x)
6. **Aggregate All Cities** - Combines into single digest
7. **Send Email** - Delivers formatted report

## ⚙️ Configuration

**Add/Remove Cities:**

Edit the `cities` array in "Loop Through 5 Cities" node:
```javascript
const cities = [
  { name: 'London', country: 'GB' },
  { name: 'Your City', country: 'CODE' },
  // Add more cities here
];
```

**Alert Thresholds:**

Modify in the `config` object (in Fahrenheit):
```javascript
heatThreshold: 90,   // °F
frostThreshold: 32   // °F
```

**Change Units:**

To use Celsius instead of Fahrenheit:
- Change `units: 'imperial'` to `units: 'metric'`
- Update thresholds: `heatThreshold: 32` (°C), `frostThreshold: 0` (°C)

## 📊 Database Schema

The workflow stores weather data with these fields:
- City, temperature, condition, humidity, wind speed
- Alert type and timestamp
- Raw API response for debugging

## 🚀 Testing

1. Set Schedule Trigger to manual or frequent interval
2. Execute workflow
3. Check email inbox for digest
4. Verify Supabase table has 5 new rows (one per city)

## 📧 Email Output

The digest includes:
- Header with date and alert count
- Individual cards for each city showing:
  - Temperature and feels-like
  - Weather condition
  - Humidity and wind speed
  - Sunrise/sunset times
  - Color-coded alerts when applicable

## 🔧 Troubleshooting

**No email received:**
- Check SMTP credentials in Send Email node
- Verify email isn't in spam folder

**Only 1 city showing:**
- Verify "Extract City Info" is set to "Run Once for Each Item"
- Check that "Loop Through 5 Cities" outputs 5 items

**Country showing "undefined":**
- Ensure cities array includes country codes
- Verify API response contains `sys.country`

**Database errors:**
- Check Supabase key has write permissions
- Verify table schema matches the SQL above

## 📝 License

MIT License - Feel free to modify and use for your own projects.

## 🙏 Acknowledgments

- Weather data provided by [OpenWeatherMap](https://openweathermap.org)
- Database hosting by [Supabase](https://supabase.com)
- Workflow automation by [n8n](https://n8n.io)
