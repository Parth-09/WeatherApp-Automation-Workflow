# WeatherApp-Automation-Workflow
# 🌤️ Daily Weather Automation with n8n

An automated weather monitoring system that fetches daily weather data for multiple cities, analyzes conditions, stores historical data in Supabase, and delivers a beautiful consolidated email digest.

## ✨ Features

- ✅ Multi-city weather monitoring (5 cities by default)
- ✅ Intelligent weather alerts (precipitation, heat, frost, wind)
- ✅ Data persistence in Supabase database
- ✅ Beautiful HTML email digest with all cities
- ✅ Automatic daily scheduling
- ✅ Error handling with retry logic

## 🛠️ Tech Stack

- **n8n** - Workflow automation
- **OpenWeatherMap API** - Weather data
- **Supabase** - PostgreSQL database
- **Email (SMTP)** - Notification delivery

## 📋 Prerequisites

- n8n account (cloud or self-hosted)
- OpenWeatherMap API key
- Supabase account
- Email service (Gmail/SMTP)

## 🚀 Setup Instructions

### 1. OpenWeatherMap API

1. Sign up at [openweathermap.org](https://openweathermap.org/api)
2. Get your API key from the dashboard
3. Copy the key for workflow configuration

### 2. Supabase Database

Create a new Supabase project and run this SQL:
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

Get your project URL and API key from Settings > API.

### 3. Email Configuration

**For Gmail:**
- Enable 2FA in your Google account
- Generate an App Password
- Use `smtp.gmail.com`, port `465` (SSL)

### 4. Import Workflow

1. Download `weather_automation.json` from this repo
2. In n8n, click **Import from File**
3. Configure credentials:
   - OpenWeatherMap API key
   - Supabase URL and anon key
   - Email SMTP credentials

### 5. Customize Cities

Edit the "Loop Through 5 Cities" node to add/remove cities:
```javascript
const cities = [
  { name: 'London', country: 'GB' },
  { name: 'New York', country: 'US' },
  { name: 'Tokyo', country: 'JP' },
  { name: 'Sydney', country: 'AU' },
  { name: 'Paris', country: 'FR' }
];
```

### 6. Set Schedule

Default: Daily at 8:00 AM  
Cron: `0 8 * * *`

For testing, use: `*/5 * * * *` (every 5 minutes)

## 📊 Workflow Structure
```
Schedule Trigger
    ↓
Loop Through 5 Cities (outputs 5 items)
    ↓
Fetch Weather from API (loops 5x)
    ↓
Extract & Analyze Data (loops 5x)
    ↓
Save to Supabase (loops 5x)
    ↓
Aggregate All Cities (combines to 1 item)
    ↓
Send Email Digest (1 email with all cities)
```

## 🎯 Alert Rules

- **Precipitation:** Rain, snow, drizzle, storm, thunder
- **Heat:** Temperature > 90°F (32°C if using metric)
- **Frost:** Temperature < 32°F (0°C if using metric)
- **Wind:** Wind speed > 30 mph (50 km/h if using metric)

## 🔧 Configuration

**Units:** Set in "Loop Through 5 Cities" node
- `units: 'imperial'` for Fahrenheit/mph
- `units: 'metric'` for Celsius/km/h

**Email Recipient:** Set in "Send Email" node

**Schedule:** Set in "Schedule Trigger" node

## 📸 Screenshots

![Workflow Overview](workflow_screenshot.png)

## 🐛 Troubleshooting

**Only 1 city showing:**
- Check "Extract City Info" mode is "Run Once for Each Item"
- Verify each node shows 5 items output (except Aggregate)

**Country showing "undefined":**
- Ensure cities include country codes in config
- Check OpenWeatherMap API response includes `sys.country`

**Alerts not working:**
- Verify temperature thresholds match your units (F vs C)
- Check alert conditions in "Extract City Info" node

**Email not sending:**
- Verify SMTP credentials
- Check firewall/port settings
- Ensure recipient email is valid

## 📝 License

MIT

## 🤝 Contributing

Feel free to submit issues and pull requests!

## 💡 Future Enhancements

- 5-day weather forecast
- SMS notifications via Twilio
- Weather trend analysis
- Custom alert thresholds per city
- Dashboard visualization

---

Built with ❤️ using n8n workflow automation
