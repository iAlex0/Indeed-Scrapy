# Indeed-Scraper
Python Scrapy spider that scrapes Jobs data from [Indeed.com](https://www.indeed.com/). There are two versions:

1. **Scrapes Job Summary Data: (indeed_search)** The scraper will query the Indeed search page with your query parameters and extract the job data directly from the search results.
2. **Scrapes Full Job Data: (indeed_jobs)** The scraper will crawl the Indeed search pages with your query parameters, then send a request to each individual job page and scrape all the job data from the page.

Both of these scrapers only scrape some of the available data, however, you can easily expand them to scrape other data that is available in the response.

These scrapers extract the following fields from Indeed jobs pages:

- Company Name
- Company Location
- Job Title
- Job Description
- Job Salary
- Job Location
- Etc.

<br>
---------------------------------------
<br>


## Getting Started
These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

1. Clone this project: `git clone https://github.com/iAlex0/Indeed-Scrapy.git`
2. Create a Python Virtual Environment: `python3 -m venv venv`
3. Activate the Python Virtual Environment: `venv/Scripts/activate`
4. Install the project requirements: `pip install -r requirements.txt`
5. Change working directory to the project folder: `cd indeed`
6. Add your API key to the settings.py file:

    - SCRAPEOPS_API_KEY = 'YOUR_API_KEY'

7. Run the following command: `scrapy list` 
    - Returns the following:

        - indeed_jobs
        - indeed_search

<br>

## ScrapeOps Proxy
This Indeed spider uses [ScrapeOps Proxy](https://scrapeops.io/proxy-aggregator/) as the proxy solution. ScrapeOps has a free plan that allows you to make up to 1,000 requests per month which makes it ideal for the development phase, but can be easily scaled up to millions of pages per month if needs be.

You can [sign up for a free API key here](https://scrapeops.io/app/register/main).

<br>
---------------------------------------
<br>

## Running The Scrapers

To run the Indeed spiders, first set the job query parameters you want to search by updating the `keyword_list` and `location_list` in the spiders.

For example:
```python

def start_requests(self):
    keyword_list = ['software engineer', 'devops engineer', 'product manager']
    location_list = ['California', 'texas']
    for keyword in keyword_list:
        for location in location_list:
            indeed_jobs_url = self.get_indeed_search_url(keyword, location)
            yield scrapy.Request(url=indeed_jobs_url, callback=self.parse_search_results, meta={'keyword': keyword, 'location': location, 'offset': 0})

```
<br>
Then to run the spiders, enter one of the following commands:

| Spider  |      Command      |
|----------|-------------|
| **Job Summary Data** |  `scrapy crawl indeed_search` | 
| **Full Job Data** |  `scrapy crawl indeed_jobs` | 

<br>

### Extract More/Different Data
The JSON blobs the spiders extract the job data from are pretty big so the spiders are configured to only parse some of the data. 

You can expand or change the data that gets extract by changing the yield statements:

```python

yield {
        'keyword': keyword,
        'location': location,
        'page': round(offset / 10) + 1 if offset > 0 else 1,
        'position': index,
        'company': job.get('company'),
        'companyRating': job.get('companyRating'),
        'companyReviewCount': job.get('companyReviewCount'),
        'companyRating': job.get('companyRating'),
        'highlyRatedEmployer': job.get('highlyRatedEmployer'),
        'jobkey': job.get('jobkey'),
        'jobTitle': job.get('title'),
        'jobLocationCity': job.get('jobLocationCity'),
        'jobLocationPostal': job.get('jobLocationPostal'),
        'jobLocationState': job.get('jobLocationState'),
        'maxSalary': job.get('estimatedSalary').get('max') if job.get('estimatedSalary') is not None else 0,
        'minSalary': job.get('estimatedSalary').get('min') if job.get('estimatedSalary') is not None else 0,
        'salaryType': job.get('estimatedSalary').get('max') if job.get('estimatedSalary') is not None else 'none',
        'pubDate': job.get('pubDate'),
    }
```
<br>
---------------------------------------
<br>

### Speeding Up The Crawl
The spiders are set to only use 1 concurrent thread in the settings.py file as the ScrapeOps Free Proxy Plan only gives you 1 concurrent thread.

However, if you upgrade to a paid ScrapeOps Proxy plan you will have more concurrent threads. Then you can increase the concurrency limit in your scraper by updating the `CONCURRENT_REQUESTS` value in your ``settings.py`` file.

```python
# settings.py

CONCURRENT_REQUESTS = 32
```
<br>
---------------------------------------
<br>


### Storing Data
The spiders are set to save the scraped data in a JSONl file and store it in a data folder using [Scrapy's Feed Export functionality](https://docs.scrapy.org/en/latest/topics/feed-exports.html).

```python

custom_settings = {
        'FEEDS': { 'data/%(name)s_%(time)s.jsonl': { 'format': 'jsonl',}}
        }

```
<br>
If you would like to save your CSV files to a csv file update the custom_settings in your spider file:

For example:

```python

custom_settings = {
        'FEEDS': { 'data/%(name)s_%(time)s.csv': { 'format': 'csv',}}
        }

```
<br>
---------------------------------------
<br>

## License
MIT © iAlex0

