import requests

def hello():
    return "Hello from the weather tool!"

def get_weather(city: str) -> str:
    """
    מקבל שם עיר (עברית/אנגלית), ומחזיר תקציר מזג אוויר קצר.
    משתמש ב-Open-Meteo (ללא מפתח API).
    """
    # שלב 1: גאוקודינג - מוצאים קואורדינטות לעיר
    geo_resp = requests.get(
        "https://geocoding-api.open-meteo.com/v1/search",
        params={"name": city, "count": 1, "language": "he", "format": "json"},
        timeout=15
    )
    geo = geo_resp.json()
    if not geo.get("results"):
        return f"לא מצאתי עיר בשם: {city}"

    place = geo["results"][0]
    lat, lon = place["latitude"], place["longitude"]
    city_name = place["name"]

    # שלב 2: מזג אוויר נוכחי
    wx_resp = requests.get(
        "https://api.open-meteo.com/v1/forecast",
        params={
            "latitude": lat,
            "longitude": lon,
            "current": "temperature_2m,wind_speed_10m,precipitation"
        },
        timeout=15
    )
    wx = wx_resp.json().get("current", {})

    t = wx.get("temperature_2m")
    w = wx.get("wind_speed_10m")
    p = wx.get("precipitation")
    ts = wx.get("time")

    return (
        f"עיר: {city_name} | זמן: {ts}\n"
        f"טמפרטורה: {t}°C | רוח: {w} m/s | משקעים: {p} מ\"מ"
    )
