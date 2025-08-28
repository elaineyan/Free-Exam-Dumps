# Free Dumps for any Professional Exam Certification 🕸️

This repository contains a Python script designed to scrape exam question links from the [Exam Topics](https://www.examtopics.com) website based on a specific provider and exam code. 
The script is optimized to run in Google Colab and uses parallel requests for efficient data fetching.

### Fetches 2.5 pages per second 📄⌛

## Features ✨

- Scrapes exam question links from Exam Topics for a specified provider.
- Filters the links based on a given exam code.
- Uses multithreading to speed up the scraping process.
- Groups and saves the exam question links in a structured text file.

## Setup Instructions 🚀

### 1. Open in Google Colab

To get started, open this script in Google Colab using the following link:

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

### 2. Install Required Libraries

Make sure to install the necessary Python libraries before running the script:

```python
!pip install requests beautifulsoup4 tqdm
```
### 3. Run the Script
Get the code from ```main.ipynb``` or ```main.py``` and run it in a Colab cell.

## Code Explanation 🧩
The script is divided into several parts, each with a specific purpose:

### Import Libraries:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup
import re
from concurrent.futures import ThreadPoolExecutor, as_completed
from tqdm import tqdm
```
- **webdriver:** For launching a Chrome browser to load examtopics page.
- **BeautifulSoup:** For parsing HTML content.
- **re:** For regular expression operations.
- **ThreadPoolExecutor:** For parallel execution of tasks.
- **tqdm:** For displaying progress bars.

### Scraper Class:
#### Initialization

```python
class Scraper:
    def __init__(self, provider):
        self.provider = provider.lower()
        self.base_url = f"https://www.examtopics.com/discussions/{self.provider}/"
        options = webdriver.ChromeOptions()
        self.driver = webdriver.Chrome(options=options)
```
- Initializes a session for making requests.
- Sets the base URL based on the provided exam provider.

#### Get Number of Pages
```python
    def get_num_pages(self):
        try:
            self.driver.get(self.base_url)
            # wait until the pagination indicator is present
            WebDriverWait(self.driver, 10).until(
                EC.presence_of_element_located((By.CLASS_NAME, "discussion-list-page-indicator"))
            )

            soup = BeautifulSoup(self.driver.page_source, "html.parser")
            container = soup.find("span", {"class": "discussion-list-page-indicator"})
            strong_tags = container.find_all("strong")

            return int(strong_tags[1].text.strip())
        except Exception as e:
            print(f"Error fetching page count: {e}")
            return 0
```
- Fetches and parses the number of pages available for the provider.

#### Fetch Page Links
```python
    def fetch_page_links(self, page, search_string):
        try:
            self.driver.get(f"{self.base_url}{page}/")

            # wait until discussion links are loaded
            WebDriverWait(self.driver, 10).until(
                EC.presence_of_all_elements_located((By.CLASS_NAME, "discussion-link"))
            )

            soup = BeautifulSoup(self.driver.page_source, "html.parser")
            discussions = soup.find_all("a", {"class": "discussion-link"})
            links = [
                d["href"].replace("/discussions", "https://www.examtopics.com/discussions", 1)
                for d in discussions if search_string in d.text
            ]
            return links
        except Exception as e:
            print(f"\nError on page {page}: {e}")
            return []
```
- Fetches and filters exam question links from a specific page.

#### Get Exam Question Links
```python
    def get_discussion_links(self, num_pages, search_string):
        links = []
        with ThreadPoolExecutor(max_workers=10) as executor:
            futures = [executor.submit(self.fetch_page_links, page, search_string) for page in range(1, num_pages + 1)]
            with tqdm(total=num_pages, desc="Fetching Links", unit="page") as pbar:
                for future in as_completed(futures):
                    page_links = future.result()
                    links.extend(page_links)
                    pbar.update(1)
        return links
```
- Uses parallel requests to fetch exam question links from multiple pages simultaneously.

### Utility Functions
#### Extract Topic and Question

```python
def extract_topic_question(link):
    match = re.search(r'topic-(\d+)-question-(\d+)', link)
    return (int(match.group(1)), int(match.group(2))) if match else (None, None)
```
- Extracts topic and question numbers from the exam question link.

#### Write Grouped Links to File
```python
def write_grouped_links_to_file(filename, links):
    grouped_links = {}
    for link in sorted(links, key=extract_topic_question):
        topic, question = extract_topic_question(link)
        grouped_links.setdefault(topic, []).append(link)

    with open(filename, 'w') as f:
        for topic, links in grouped_links.items():
            f.write(f'Topic {topic}:\n')
            for link in links:
                f.write(f' - {link}\n')
            print(f"Topic {topic} links added to file.")
```
- Groups and writes the exam question links to a file based on their topic.

### Main Function:

```python
def main():
    provider = input("Enter provider name: ")
    scraper = Scraper(provider)
    num_pages = scraper.get_num_pages()
    print("Total Pages:", num_pages)
    if num_pages > 0:
        search_string = input("Enter exam code or enter QUIT to exit: ").upper()
        if search_string != 'QUIT':
            links = scraper.get_discussion_links(num_pages, search_string)
            filename = f'{search_string} dumps.txt'
            print(f"\nYour file will be named {filename}")
            write_grouped_links_to_file(filename, links)
            print("File generation complete.")
        else:
            return
    else:
        print("No pages found for the provider.")

if __name__ == "__main__":
    main()
```

- Prompts the user for the provider and exam code (put the correct exam code).
- Fetches and processes the exam question links.
- Saves the links to a text file.
  
![image](https://github.com/user-attachments/assets/5a896479-5f7d-4904-a8e8-e0371d7a8a9c)


# License 📄
### This project is licensed under the MIT License.
