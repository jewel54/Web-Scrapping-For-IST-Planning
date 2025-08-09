import wikipedia
import requests
from bs4 import BeautifulSoup
import openpyxl
import time
import logging
from datetime import datetime
import re

# Set up logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('cabinet_scraper_2025.log'),
        logging.StreamHandler()
    ]
)

# Full list of 193 UN-recognized countries
countries = [
    "Afghanistan", "Albania", "Algeria", "Andorra", "Angola", "Antigua and Barbuda", "Argentina",
    "Armenia", "Australia", "Austria", "Azerbaijan", "Bahamas", "Bahrain", "Bangladesh", "Barbados",
    "Belarus", "Belgium", "Belize", "Benin", "Bhutan", "Bolivia", "Bosnia and Herzegovina", "Botswana",
    "Brazil", "Brunei", "Bulgaria", "Burkina Faso", "Burundi", "Cabo Verde", "Cambodia", "Cameroon",
    "Canada", "Central African Republic", "Chad", "Chile", "China", "Colombia", "Comoros", "Congo (Congo-Brazzaville)",
    "Costa Rica", "Croatia", "Cuba", "Cyprus", "Czech Republic", "Democratic Republic of the Congo", "Denmark",
    "Djibouti", "Dominica", "Dominican Republic", "Ecuador", "Egypt", "El Salvador", "Equatorial Guinea",
    "Eritrea", "Estonia", "Eswatini", "Ethiopia", "Fiji", "Finland", "France", "Gabon", "Gambia", "Georgia",
    "Germany", "Ghana", "Greece", "Grenada", "Guatemala", "Guinea", "Guinea-Bissau", "Guyana", "Haiti",
    "Honduras", "Hungary", "Iceland", "India", "Indonesia", "Iran", "Iraq", "Ireland", "Israel", "Italy",
    "Ivory Coast", "Jamaica", "Japan", "Jordan", "Kazakhstan", "Kenya", "Kiribati", "Kuwait", "Kyrgyzstan",
    "Laos", "Latvia", "Lebanon", "Lesotho", "Liberia", "Libya", "Liechtenstein", "Lithuania", "Luxembourg",
    "Madagascar", "Malawi", "Malaysia", "Maldives", "Mali", "Malta", "Marshall Islands", "Mauritania",
    "Mauritius", "Mexico", "Micronesia", "Moldova", "Monaco", "Mongolia", "Montenegro", "Morocco",
    "Mozambique", "Myanmar", "Namibia", "Nauru", "Nepal", "Netherlands", "New Zealand", "Nicaragua", "Niger",
    "Nigeria", "North Korea", "North Macedonia", "Norway", "Oman", "Pakistan", "Palau", "Palestine", "Panama",
    "Papua New Guinea", "Paraguay", "Peru", "Philippines", "Poland", "Portugal", "Qatar", "Romania", "Russia",
    "Rwanda", "Saint Kitts and Nevis", "Saint Lucia", "Saint Vincent and the Grenadines", "Samoa",
    "San Marino", "Sao Tome and Principe", "Saudi Arabia", "Senegal", "Serbia", "Seychelles", "Sierra Leone",
    "Singapore", "Slovakia", "Slovenia", "Solomon Islands", "Somalia", "South Africa", "South Korea",
    "South Sudan", "Spain", "Sri Lanka", "Sudan", "Suriname", "Sweden", "Switzerland", "Syria", "Taiwan",
    "Tajikistan", "Tanzania", "Thailand", "Timor-Leste", "Togo", "Tonga", "Trinidad and Tobago", "Tunisia",
    "Turkey", "Turkmenistan", "Tuvalu", "Uganda", "Ukraine", "United Arab Emirates", "United Kingdom",
    "United States", "Uruguay", "Uzbekistan", "Vanuatu", "Vatican City", "Venezuela", "Vietnam", "Yemen",
    "Zambia", "Zimbabwe"
]

class CabinetScraper2025:
    def __init__(self, output_file="current_officials_2025.xlsx"):
        self.output_file = output_file
        self.wb = openpyxl.Workbook()
        self.ws = self.wb.active
        self.ws.title = "Current Officials 2025"
        self.setup_excel_headers()
        self.success_count = 0
        self.error_count = 0
        
        # Keywords for filtering high-level positions
        self.position_keywords = [
            'prime minister', 'president', 'minister', 'secretary of state',
            'chancellor', 'premier', 'chief minister', 'deputy prime minister',
            'vice president', 'speaker', 'chief justice', 'attorney general'
        ]
        
        # Current/recent indicators
        self.current_indicators = [
            '2025', '2024', 'current', 'incumbent', 'serving', 'since', 'from'
        ]
        
    def setup_excel_headers(self):
        """Set up Excel headers - removed date column"""
        headers = ["Country", "Official Name", "Position/Title", "Source URL"]
        self.ws.append(headers)
        
        # Format headers
        for cell in self.ws[1]:
            cell.font = openpyxl.styles.Font(bold=True)
            cell.fill = openpyxl.styles.PatternFill(start_color="E6E6FA", 
                                                  end_color="E6E6FA", 
                                                  fill_type="solid")

    def clean_text(self, text):
        """Clean and normalize text data"""
        if not text:
            return ""
        # Remove extra whitespace and newlines
        text = re.sub(r'\s+', ' ', text.strip())
        # Remove reference markers like [1], [2], etc.
        text = re.sub(r'\[\d+\]', '', text)
        # Remove parenthetical dates unless they contain 2024/2025
        text = re.sub(r'\([^)]*(?<!202[45])[^)]*\)', '', text)
        return text.strip()

    def is_high_level_position(self, position):
        """Check if position is high-level (minister, president, PM, etc.)"""
        position_lower = position.lower()
        return any(keyword in position_lower for keyword in self.position_keywords)

    def is_current_official(self, text):
        """Check if the text indicates current/recent service"""
        text_lower = text.lower()
        
        # Look for 2024/2025 dates
        if re.search(r'202[45]', text):
            return True
            
        # Look for current service indicators
        if any(indicator in text_lower for indicator in self.current_indicators):
            return True
            
        # Avoid past tense indicators
        past_indicators = ['former', 'ex-', 'until', 'ended', 'resigned', 'died', 'retired']
        if any(past in text_lower for past in past_indicators):
            return False
            
        return True

    def get_cabinet_data(self, country):
        """Enhanced cabinet data extraction focusing on current high-level officials"""
        try:
            # Multiple search terms to try
            search_terms = [
                f"Politics of {country}",
                f"Government of {country}",
                f"Cabinet of {country}",
                f"{country} government",
                f"List of heads of government of {country}",
                f"President of {country}",
                f"Prime Minister of {country}"
            ]
            
            page_found = False
            page_url = None
            
            for search_term in search_terms:
                try:
                    results = wikipedia.search(search_term, results=5)
                    if results:
                        for result in results:
                            try:
                                page = wikipedia.page(result, auto_suggest=False)
                                page_url = page.url
                                page_found = True
                                logging.info(f"Found page for {country}: {result}")
                                break
                            except wikipedia.exceptions.DisambiguationError as e:
                                # Try the first option from disambiguation
                                try:
                                    page = wikipedia.page(e.options[0], auto_suggest=False)
                                    page_url = page.url
                                    page_found = True
                                    logging.info(f"Found disambiguated page for {country}: {e.options[0]}")
                                    break
                                except:
                                    continue
                            except:
                                continue
                        if page_found:
                            break
                except:
                    continue
            
            if not page_found:
                logging.warning(f"No Wikipedia page found for {country}")
                return False

            # Scrape the page
            response = requests.get(page_url, timeout=15)
            response.raise_for_status()
            soup = BeautifulSoup(response.content, "html.parser")

            # Look for current officials in multiple ways
            officials_found = False
            
            # Method 1: Extract from infoboxes (most reliable for current officials)
            if self.extract_from_infoboxes(soup, country, page_url):
                officials_found = True
            
            # Method 2: Extract from government tables
            if self.extract_from_government_tables(soup, country, page_url):
                officials_found = True
                
            # Method 3: Extract from text patterns
            if self.extract_current_officials_from_text(soup, country, page_url):
                officials_found = True
            
            if officials_found:
                self.success_count += 1
                return True
            else:
                logging.warning(f"No current officials found for {country}")
                return False
            
        except Exception as e:
            logging.error(f"Error processing {country}: {str(e)}")
            self.error_count += 1
            return False

    def extract_from_infoboxes(self, soup, country, url):
        """Extract current officials from Wikipedia infoboxes"""
        infoboxes = soup.find_all("table", {"class": ["infobox", "vcard"]})
        data_found = False
        
        for infobox in infoboxes:
            rows = infobox.find_all("tr")
            
            for row in rows:
                th = row.find("th")
                td = row.find("td")
                
                if th and td:
                    position = self.clean_text(th.get_text())
                    official = self.clean_text(td.get_text())
                    
                    if (self.is_high_level_position(position) and 
                        official and 
                        len(official) > 2 and
                        self.is_current_official(official)):
                        
                        # Clean up the official name (remove extra info)
                        official_clean = re.split(r'\((?!.*202[45])', official)[0].strip()
                        
                        self.ws.append([country, official_clean, position, url])
                        data_found = True
                        logging.info(f"Found official for {country}: {official_clean} - {position}")
        
        return data_found

    def extract_from_government_tables(self, soup, country, url):
        """Extract current officials from government/cabinet tables"""
        tables = soup.find_all("table", {"class": "wikitable"})
        data_found = False
        
        for table in tables:
            rows = table.find_all("tr")
            if len(rows) < 2:
                continue
                
            # Check if this looks like a current government table
            table_text = table.get_text().lower()
            if not any(keyword in table_text for keyword in 
                      ['current', '2025', '2024', 'incumbent', 'government']):
                continue
                
            # Try to identify column structure
            header_row = rows[0]
            headers = [th.get_text().lower().strip() for th in header_row.find_all(["th", "td"])]
            
            position_col = -1
            name_col = -1
            
            for i, header in enumerate(headers):
                if any(word in header for word in ['position', 'office', 'title', 'ministry']):
                    position_col = i
                elif any(word in header for word in ['name', 'minister', 'official', 'incumbent']):
                    name_col = i
            
            # Extract data from table rows
            for row in rows[1:]:
                cols = row.find_all(["td", "th"])
                if len(cols) < 2:
                    continue
                
                # Get position and name based on identified columns
                if position_col >= 0 and position_col < len(cols):
                    position = self.clean_text(cols[position_col].get_text())
                else:
                    position = self.clean_text(cols[0].get_text())
                
                if name_col >= 0 and name_col < len(cols):
                    official = self.clean_text(cols[name_col].get_text())
                else:
                    official = self.clean_text(cols[1].get_text() if len(cols) > 1 else "")
                
                # Check if this is a high-level current position
                row_text = row.get_text()
                if (self.is_high_level_position(position) and 
                    official and 
                    len(official) > 2 and
                    self.is_current_official(row_text)):
                    
                    # Clean up the official name
                    official_clean = re.split(r'\((?!.*202[45])', official)[0].strip()
                    
                    self.ws.append([country, official_clean, position, url])
                    data_found = True
                    logging.info(f"Found official for {country}: {official_clean} - {position}")
        
        return data_found

    def extract_current_officials_from_text(self, soup, country, url):
        """Extract current officials from page text using pattern matching"""
        data_found = False
        
        # Look for recent paragraphs mentioning current officials
        paragraphs = soup.find_all('p')
        
        for p in paragraphs:
            text = p.get_text()
            
            # Skip if paragraph doesn't seem current
            if not self.is_current_official(text):
                continue
            
            # Pattern for "Position: Name" or "Name is the Position"
            patterns = [
                r'(Prime Minister|President|Minister of [^,\n]+):\s*([A-Z][a-zA-Z\s]+?)(?:\s*\(|,|\.|\n|$)',
                r'([A-Z][a-zA-Z\s]+?)\s+(?:is|serves as|holds the office of)\s+(?:the\s+)?(Prime Minister|President|Minister of [^,\n]+)',
                r'(?:Current|The|Incumbent)\s+(Prime Minister|President|Minister of [^,\n]+)\s+(?:is|of [A-Za-z\s]+ is)\s+([A-Z][a-zA-Z\s]+?)(?:\s*\(|,|\.|\n|$)',
            ]
            
            for pattern in patterns:
                matches = re.findall(pattern, text, re.IGNORECASE)
                for match in matches:
                    if len(match) == 2:
                        pos1, pos2 = match
                        
                        # Determine which is position and which is name
                        if self.is_high_level_position(pos1):
                            position, official = pos1, pos2
                        else:
                            position, official = pos2, pos1
                        
                        if (official and len(official.strip()) > 2 and 
                            self.is_high_level_position(position)):
                            
                            official_clean = official.strip()
                            position_clean = position.strip()
                            
                            self.ws.append([country, official_clean, position_clean, url])
                            data_found = True
                            logging.info(f"Found official for {country}: {official_clean} - {position_clean}")
                            break
        
        return data_found

    def process_all_countries(self):
        """Process all countries with progress tracking"""
        logging.info(f"Starting to process {len(countries)} countries for current officials...")
        
        for idx, country in enumerate(countries, 1):
            logging.info(f"[{idx}/{len(countries)}] Processing {country}...")
            
            success = self.get_cabinet_data(country)
            
            if success:
                print(f"✅ {country}")
            else:
                print(f"❌ {country}")
            
            # Rate limiting - be respectful to Wikipedia
            time.sleep(3)  # Slightly longer delay for more complex scraping
            
            # Save progress every 15 countries
            if idx % 15 == 0:
                self.save_progress()
                logging.info(f"Progress saved. Processed {idx} countries so far.")

    def save_progress(self):
        """Save current progress"""
        temp_file = f"temp_{self.output_file}"
        self.wb.save(temp_file)

    def finalize(self):
        """Save final results and generate summary"""
        self.wb.save(self.output_file)
        
        total_rows = self.ws.max_row - 1  # Subtract header row
        
        summary = f"""
=== CURRENT OFFICIALS 2025 SCRAPING SUMMARY ===
Output file: {self.output_file}
Countries processed: {len(countries)}
Successful extractions: {self.success_count}
Errors encountered: {self.error_count}
Total current officials found: {total_rows}
Success rate: {(self.success_count/len(countries)*100):.1f}%

Data includes only:
- Current Ministers, Prime Ministers, Presidents
- Officials serving in 2025
- High-level government positions only
- Source URLs for verification

Filters applied:
- Only high-level positions (Minister+, President, PM, etc.)
- Only current/recent officials (2024-2025)
- Excluded former/past officials
        """
        
        print(summary)
        logging.info(summary)
        
        return summary

def main():
    """Main execution function"""
    print("🚀 Starting Current Officials 2025 Scraper...")
    print("This will collect information on current high-level government officials")
    print("Focus: Ministers, Prime Ministers, Presidents currently serving in 2025")
    print("Data source: Wikipedia")
    print("Please be patient - this process respects rate limits and may take some time.\n")
    
    scraper = CabinetScraper2025()
    
    try:
        scraper.process_all_countries()
        summary = scraper.finalize()
        
        print(f"\n✅ Scraping completed! Check '{scraper.output_file}' for results.")
        
    except KeyboardInterrupt:
        print("\n⚠️ Process interrupted by user. Saving current progress...")
        scraper.save_progress()
        print("Progress saved to temporary file.")
        
    except Exception as e:
        logging.error(f"Unexpected error: {str(e)}")
        print(f"❌ An error occurred: {str(e)}")
        scraper.save_progress()

if __name__ == "__main__":
    main()
