# Assessment-for-python-
//code for handling data
import time
import json
import pandas as pd
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.action_chains import ActionChains
from bs4 import BeautifulSoup

# Set up Selenium WebDriver
chrome_options = Options()
chrome_options.add_argument("--headless")  # Run in headless mode
service = Service('path/to/chromedriver')  # Update with the path to your chromedriver
driver = webdriver.Chrome(service=service, options=chrome_options)

# List of LinkedIn URLs to scrape
urls = [
    "https://www.linkedin.com/jobs/search?location=India&geoId=102713980&f_C=1035&position=1&pageNum=0",
    "https://www.linkedin.com/jobs/search?keywords=&location=India&geoId=102713980&f_C=1441",
    "https://www.linkedin.com/jobs/search?keywords=&location=India&geoId=102713980&f_TPR=r86400&f_C=1586&position=1&pageNum=0"
]

# Data storage
jobs_data = []

def parse_job_element(job_element):
    try:
        job = {}
        job['company'] = job_element.find('h4', class_='base-search-card__subtitle').text.strip()
        job['job_title'] = job_element.find('h3', class_='base-search-card__title').text.strip()
        job['linkedin_job_id'] = job_element.find('a', class_='base-card__full-link')['href'].split('/')[-1]
        job['location'] = job_element.find('span', class_='job-search-card__location').text.strip()
        job['posted_on'] = job_element.find('time')['aria-label'].strip()
        job['posted_date'] = job_element.find('time')['datetime'].split('T')[0]  # Simplify date
        job['Employment type'] = job_element.find('span', class_='job-search-card__employment-type').text.strip() if job_element.find('span', class_='job-search-card__employment-type') else 'null'
        job['Seniority level'] = job_element.find('span', class_='job-search-card__seniority-level').text.strip() if job_element.find('span', class_='job-search-card__seniority-level') else 'null'
        return job
    except Exception as e:
        print(f"Error parsing job element: {e}")
        return None

for url in urls:
    driver.get(url)
    time.sleep(5)  # Wait for the page to load

    soup = BeautifulSoup(driver.page_source, 'html.parser')
    job_elements = soup.find_all('li', class_='job-search-card')
    
    for job_element in job_elements:
        job = parse_job_element(job_element)
        if job:
            jobs_data.append(job)

driver.quit()

# Save data to JSON and CSV
with open('jobs_data.json', 'w') as json_file:
    json.dump(jobs_data, json_file, indent=4)

df = pd.DataFrame(jobs_data)
df.to_csv('jobs_data.csv', index=False)

print("Data scraping completed and saved to jobs_data.json and jobs_data.csv")
