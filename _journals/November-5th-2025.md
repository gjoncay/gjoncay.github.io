---
title: "November 5th, 2025"
author: Grant Oncay
tags:
  - Python
---


Messing around with APIs today. I put a lot of time into Angela Yu's 100 Days of Python... it's a great course, but some of the projects really make me want to step on a lego. I needed to break away from the course for a bit and just make sure I have the fundamentals (mostly of `requests` and environment variables) down.

Knocked out a sample just to prove I could work through pulling data from Virus Total without leaning on anything besides the documentation.

```
from dotenv import load_dotenv
import os
import pprint
import requests
load_dotenv()
pp = pprint.PrettyPrinter(indent=4)
BASE_URL = "https://www.virustotal.com/api/v3/"
ENDPOINT = "files/{id}"
VT_API_KEY = os.getenv("VT_API_KEY")
headers = {'accept': 'application/json', 'x-apikey': VT_API_KEY}


# File report with hash input
def get_file_report(file_hash):
    template_url = BASE_URL + ENDPOINT
    final_url = template_url.replace("{id}", file_hash)
    response = requests.get(final_url, headers=headers)
    response.raise_for_status()
    return response.json()

data = get_file_report("158c0607af02d9fe9331faa9cabf5d39")
pp.pprint(data)
```
