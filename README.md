import os
import shutil
import requests
from bs4 import BeautifulSoup
import smtplib
from email.mime.text import MIMEText
import pandas as pd
import schedule
import time

# Function to organize files
def organize_files(directory):
    os.makedirs(os.path.join(directory, 'TextFiles'), exist_ok=True)
    os.makedirs(os.path.join(directory, 'Images'), exist_ok=True)
    
    for filename in os.listdir(directory):
        if filename.endswith('.txt'):
            shutil.move(os.path.join(directory, filename), os.path.join(directory, 'TextFiles', filename))
        elif filename.endswith('.jpg') or filename.endswith('.png'):
            shutil.move(os.path.join(directory, filename), os.path.join(directory, 'Images', filename))

# Function to scrape a website
def scrape_website(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    headlines = [item.get_text() for item in soup.find_all('h2')]
    return headlines

# Function to send an email
def send_email(subject, body, to_email):
    from_email = 'your_email@example.com'
    password = 'your_password'
    
    msg = MIMEText(body)
    msg['Subject'] = subject
    msg['From'] = from_email
    msg['To'] = to_email
    
    with smtplib.SMTP('smtp.example.com', 587) as server:
        server.starttls()
        server.login(from_email, password)
        server.send_message(msg)

# Function to analyze data
def analyze_data(file_path):
    df = pd.read_csv(file_path)
    summary = df.describe()
    summary.to_csv('summary.csv')
    return summary

# Job function for scheduling
def job():
    print("Scheduled task running...")
    organize_files('/path/to/your/directory')
    headlines = scrape_website('https://example.com')
    print("Scraped headlines:", headlines)
    send_email('Scraping Complete', 'Headlines have been scraped successfully.', 'recipient@example.com')
    analyze_data('data.csv')
    print("Data analysis complete.")

# Schedule the job every hour
schedule.every(1).hours.do(job)

if __name__ == "__main__":
    print("Starting task automation...")
    while True:
        schedule.run_pending()
        time.sleep(1)
