CabinetScraper2025

Overview

CabinetScraper2025 is a Python script that collects data on current high-level government officials (e.g., Presidents, Prime Ministers, Ministers) for 193 UN-recognized countries using Wikipedia. It features asynchronous scraping, caching, multiple output formats (Excel, CSV, JSON), and visualizations. The script is designed for researchers, journalists, and policymakers needing up-to-date political data.

What It Does

Scrapes Wikipedia: Retrieves data on current officials from pages like "Politics of [Country]" or "Cabinet of [Country]".
Extracts Key Information: Captures official names, positions, source URLs, extraction dates, and confidence scores.
Outputs Data: Saves results to Excel, CSV, or JSON with a CLI for customization.
Visualizes Data: Generates a bar chart of officials per country (officials_by_country.png).
Ensures Efficiency: Uses aiohttp for async requests, caching to reduce server load, and rate limiting to respect Wikipedia's policies.



Logs Progress: Outputs logs to cabinet_scraper_2025.log for debugging.

Installation
Install Python: Requires Python 3.6+ (tested on 3.9).

Install Dependencies:
pip install wikipedia-api aiohttp requests beautifulsoup4 openpyxl matplotlib

Save the Script: Copy the script to cabinet_scraper_2025.py
Usage

Run the script from the command line:

python cabinet_scraper_2025.py

Options
--countries: Specify countries (e.g., --countries France Germany).
--output: Set output file name (e.g., --output officials.csv).
--format: Choose output format (excel, csv, json).

Example:

python cabinet_scraper_2025.py --countries "United States" Canada --format json

Outputs
Data File: current_officials_2025.xlsx (or .csv, .json).
Visualization: officials_by_country.png (bar chart of officials per country).
Log File: cabinet_scraper_2025.log.
Cache: Stores fetched pages in cache/ directory.

Future Opportunities
To enhance the script for broader adoption and impact:
Multi-Source Integration: Add government websites, NewsAPI, or CIA World Leaders database for cross-verification.
GUI: Develop a tkinter or PyQt interface for non-technical users.
Cloud Deployment: Host on AWS Lambda or Heroku for automated, scalable scraping.
Database Support: Store data in SQLite or PostgreSQL for querying and analysis
Advanced Visualizations: Add interactive charts using plotly or web-based dashboards.
Multilingual Support: Scrape non-English Wikipedia pages for better coverage.
API: Create a REST API with FastAPI for integration with other applications.
Unit Tests: Add pytest tests for reliability.
Open-Source: Host on GitHub to encourage community contributions.

Ethical Considerations
Respects Wikipedia’s rate limits with a 0.3-second delay between request batches.
Includes user-agent header for transparency.
Attributes data to Wikipedia (CC BY-SA 3.0) in outputs.
License

MIT License. See LICENSE file for details.
