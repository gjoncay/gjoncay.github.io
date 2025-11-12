---
title: "November 12th, 2025"
author: Grant Oncay
---

I'm still trying to get as much daily practice with Python as I can. Angela Yu's 100 Days of Python on Udemy is a great course. However, a lot of the projects are broken; for example, some of the web scraping targets are paywalled or no longer have valid URLs. As such, I've tried to incorporate the same concepts into personal projects but with content I am more interested in.

Here is a snipped from today, where I grabbed the top passing leaders in the NFL with BeautifulSoup after the user inputs a year, then combined that with their actual yard count to create a dictionary of players and yards thrown for the key:value pairs.

```Python
from bs4 import BeautifulSoup
import requests

year = input("Enter a year in the format 'YYYY': ")
base_url = "https://www.nfl.com/stats/player-stats/category/passing/year/reg/all/passingyards/desc"
url = base_url.replace("year", year)
header = {
    "User-Agent": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:144.0) Gecko/20100101 Firefox/144.0"
}
response = requests.get(url, headers=header)
soup = BeautifulSoup(response.content, "html.parser")
players = soup.find_all("a", class_="d3-o-player-fullname nfl-o-cta--link")

players_list = [player.get_text(strip=True) for player in players]

yards = soup.find_all(class_="selected")
yards_list = [yard.get_text(strip=True) for yard in yards]

player_yards = dict(zip(players_list, yards_list))
print(player_yards)
```

I'll probably continue to do my own projects that use the same concepts and fundamentals taught in her course just to keep it from getting stale and to get more practice.

I'd like to focus on projects that revolve around cybersecurity as much as possible... but sometimes it's good to break it up (especially mid NFL season).