---
name: dijon
description: Gives the current weather in Dijon, France. Use when the user asks about the weather in Dijon.
allowed-tools: Bash(curl *)
---

# Weather in Dijon

1. Fetch the current conditions:
   `curl -s -m 10 "wttr.in/Dijon,France?format=j1"`
2. Reply in French with a short summary: conditions, temperature and feels-like temperature (°C), humidity, wind, and a one-line outlook for the rest of the day.
3. If the request fails or times out, say so instead of guessing.
