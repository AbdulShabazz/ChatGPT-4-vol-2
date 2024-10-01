## OpenAI ChatGPT-1-Oh-mini (Strawberry-mini version 1.2024.268)

**Q:** Can you write a Python script to clone the Unreal Engine documentation at URL, https://dev.epicgames.com/documentation/en-us/unreal-engine, along with same-domain only embedded URLs, to the local hard disk and then convert the HTML to MARKDOWN, for the purpose of developing an Unreal Engine plugin with in-engine (treeview based) documentation ?

**A:** Certainly! Below is a Python script that accomplishes the following:

	1.	Crawls the Unreal Engine Documentation Website: It starts from the specified URL (https://dev.epicgames.com/documentation/en-us/unreal-engine) and recursively downloads all pages within the same domain.
	2.	Downloads HTML Content: Saves the HTML content of each page to the local filesystem, maintaining the website’s directory structure.
	3.	Converts HTML to Markdown: Uses the markdownify library to convert the downloaded HTML files into Markdown format.
	4.	Handles Same-Domain URLs Only: Ensures that only URLs within the dev.epicgames.com domain are processed to avoid external links.

Prerequisites

Before running the script, ensure you have the following Python packages installed:

	•	requests: For making HTTP requests.
	•	beautifulsoup4: For parsing HTML content.
	•	markdownify: For converting HTML to Markdown.
	•	tqdm: For displaying progress bars (optional but recommended).

You can install them using pip:

```python
pip install requests beautifulsoup4 markdownify tqdm
```

The Python Script  

```python
import os
import re
import requests
from urllib.parse import urljoin, urlparse
from bs4 import BeautifulSoup
from markdownify import markdownify as md
from tqdm import tqdm

# Configuration
START_URL = "https://dev.epicgames.com/documentation/en-us/unreal-engine"
DOMAIN = "dev.epicgames.com"
OUTPUT_DIR = "unreal_engine_docs"

# Initialize sets to keep track of visited URLs and URLs to visit
visited = set()
to_visit = set([START_URL])

# Function to sanitize and create local file paths
def get_local_path(url):
    parsed = urlparse(url)
    path = parsed.path
    if path.endswith("/"):
        path += "index.html"
    elif not os.path.splitext(path)[1]:
        path += "/index.html"
    local_path = os.path.join(OUTPUT_DIR, parsed.netloc, path.lstrip("/"))
    return local_path

# Function to save HTML content
def save_html(content, path):
    os.makedirs(os.path.dirname(path), exist_ok=True)
    with open(path, "w", encoding='utf-8') as f:
        f.write(content)

# Function to save Markdown content
def save_markdown(html_content, path):
    markdown_content = md(html_content)
    md_path = os.path.splitext(path)[0] + ".md"
    with open(md_path, "w", encoding='utf-8') as f:
        f.write(markdown_content)

# Function to extract same-domain links from HTML
def extract_links(html, base_url):
    soup = BeautifulSoup(html, "html.parser")
    links = set()
    for tag in soup.find_all(['a', 'link', 'script', 'img']):
        attr = 'href' if tag.name in ['a', 'link'] else 'src'
        url = tag.get(attr)
        if url:
            full_url = urljoin(base_url, url)
            parsed = urlparse(full_url)
            if parsed.netloc == DOMAIN:
                # Remove URL fragments and query parameters
                clean_url = parsed.scheme + "://" + parsed.netloc + parsed.path
                links.add(clean_url)
    return links

# Main crawling loop with progress bar
print("Starting crawl...")
while to_visit:
    current_url = to_visit.pop()
    if current_url in visited:
        continue
    try:
        response = requests.get(current_url, timeout=10)
        if response.status_code != 200:
            print(f"Failed to retrieve {current_url} (Status code: {response.status_code})")
            continue
        html_content = response.text
        local_html_path = get_local_path(current_url)
        save_html(html_content, local_html_path)
        save_markdown(html_content, local_html_path)
        visited.add(current_url)
        # Extract and enqueue new links
        new_links = extract_links(html_content, current_url)
        for link in new_links:
            if link not in visited:
                to_visit.add(link)
    except Exception as e:
        print(f"Error processing {current_url}: {e}")

print("Crawling completed.")

# Optional: Create a simple tree view structure
def build_tree(start_url):
    tree = {}
    for url in visited:
        relative_path = os.path.relpath(get_local_path(url), OUTPUT_DIR)
        parts = relative_path.split(os.sep)
        current = tree
        for part in parts:
            current = current.setdefault(part, {})
    return tree

def print_tree(d, indent=0):
    for key, value in d.items():
        print('  ' * indent + key)
        if isinstance(value, dict):
            print_tree(value, indent + 1)

# Uncomment the following lines to print the directory tree
# tree = build_tree(START_URL)
# print("Directory Tree:")
# print_tree(tree)

```

Script Explanation

	1.	Configuration Section:
	   •	START_URL: The root URL to start crawling.
	   •	DOMAIN: The domain to restrict crawling within.
	   •	OUTPUT_DIR: The local directory where the documentation will be saved.
	2.	URL Handling:
	   •	The script uses urlparse to dissect URLs and ensure that only URLs within the specified domain are processed.
	   •	It sanitizes URLs to create corresponding local file paths, ensuring the directory structure mirrors the website’s structure.
	3.	Crawling Logic:
	   •	Maintains two sets: visited to keep track of already processed URLs and to_visit for URLs pending processing.
	   •	Uses requests to fetch each URL and BeautifulSoup to parse and extract same-domain links.
	   •	Saves both HTML and converted Markdown files locally.
	4.	Error Handling:
       •	The script gracefully handles HTTP errors and exceptions, logging any issues encountered during the crawl.  
	5.	Optional Tree View Structure:
	   •	At the end of the script, there’s an optional section (commented out) that builds and prints a simple tree view of the downloaded documentation. This can be useful for verifying the structure or integrating into your Unreal Engine plugin.

Usage Instructions

	1.	Set Up:
	   •	Ensure you have Python 3.x installed on your machine.
	   •	Install the required packages using pip as mentioned above.
	2.	Run the Script:
	   •	Save the script to a file, e.g., clone_unreal_docs.py.
	   •	Open a terminal or command prompt and navigate to the directory containing the script.
	   •	Execute the script:

python clone_unreal_docs.py


	3.	Output:
	   •	The documentation will be saved in the unreal_engine_docs directory (as specified by OUTPUT_DIR).
	   •	Each HTML page will have a corresponding Markdown (.md) file.
	4.	Integration with Unreal Engine Plugin:
	   •	You can use the Markdown files to build a treeview-based documentation system within your Unreal Engine plugin.
	   •	Consider parsing the Markdown files and organizing them into a hierarchical structure suitable for in-engine display.

Notes and Recommendations

	   •	Respectful Crawling: Be mindful of the website’s server load. For large documentation sites, consider adding delays between requests or limiting the number of concurrent requests.
	   •	Robots.txt Compliance: Before crawling, check the website’s robots.txt file to ensure that you’re allowed to scrape the content.
	   •	Dynamic Content: If the documentation site uses JavaScript to load content dynamically, you might need to use a headless browser like Selenium or Playwright instead of requests and BeautifulSoup.
	   •	Error Logging: For a more robust solution, consider implementing logging to capture errors and statuses instead of printing them to the console.
	   •	Incremental Updates: If the documentation is frequently updated, you might want to modify the script to support incremental updates instead of downloading everything from scratch each time.

Conclusion

This script provides a foundational approach to cloning and converting the Unreal Engine documentation for use within an Unreal Engine plugin. Depending on the complexity and specific requirements of the documentation site, you might need to enhance the script further. Nonetheless, it should serve as a solid starting point for your project.
