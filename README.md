# poc-scraping-biorxiv

## Tutorial: Scraping BioRxiv for Dengue and Arbovirus Research Articles

### Purpose:
The purpose of this tutorial is to demonstrate how to scrape research articles related to dengue and arboviruses from the BioRxiv preprint server using Python and Selenium. We will then store the results in a JSON file for further analysis.

### Step 1: Downloading Chrome WebDriver
1. Visit the Chrome WebDriver downloads page: [ChromeDriver - WebDriver for Chrome](https://googlechromelabs.github.io/chrome-for-testing/#stable).
2. Download the version of Chrome WebDriver that matches your Chrome browser version.
3. Save the downloaded WebDriver executable file to your computer.

### Step 2: Extracting and Configuring Chrome WebDriver
1. Open a terminal window.
2. Navigate to the directory where you downloaded the Chrome WebDriver executable file.
3. Extract the WebDriver executable file from the downloaded archive. You can use the `unzip` command on Unix-like systems:
   ```
   unzip chromedriver_linux64.zip
   ```
   Replace `chromedriver_linux64.zip` with the name of the downloaded archive file.
4. Once extracted, you'll have the `chromedriver` executable file.

### Step 3: Moving Chrome WebDriver to /usr/bin
1. Move the `chromedriver` executable file to the `/usr/bin` directory, which is included in the system's PATH environment variable. You may need superuser privileges to do this.
   ```
   sudo mv chromedriver /usr/bin
   ```
   You will be prompted to enter your password.

### Step 4: Making Chrome WebDriver Executable
1. After moving the `chromedriver` executable to `/usr/bin`, make it executable using the `chmod` command:
   ```
   sudo chmod +x /usr/bin/chromedriver
   ```

### Step 5: Setting up the Python Environment
1. Create a virtual environment for your Python project. Navigate to your project directory in the terminal and run:
   ```
   python3 -m venv env
   ```
2. Activate the virtual environment:
   - On Unix/Linux/macOS:
     ```
     source env/bin/activate
     ```
   - On Windows:
     ```
     .\env\Scripts\activate
     ```

### Step 6: Installing Required Libraries
1. With the virtual environment activated, install the required libraries using pip:
   ```
   pip install selenium typer
   ```

### Step 7: Writing the Python Script
1. Open a terminal window.
2. Navigate to the directory where you want to create the Python script:
   ```
   cd /path/to/your/directory
   ```
3. Use Vim or your preferred text editor to create a new Python file:
   ```
   vim scrap_biorxiv.py
   ```
4. Copy and paste the following Python script into the editor:
    ```python
    from selenium.webdriver.common.by import By
    from selenium.webdriver.common.keys import Keys

    app = typer.Typer()

    class BioRxivScraper:
        def __init__(self, driver):
            self.driver = driver
            self.url = 'https://www.biorxiv.org/'
            self.search_bar = 'input[name="keywords"]'  # CSS selector for the search input field
            self.results = []

        def navigate(self):
            self.driver.get(self.url)

        def search(self, keywords: str):
            # Wait for the page to load
            self.driver.implicitly_wait(10)

            # Find the search input field
            search_input = self.driver.find_element(By.CSS_SELECTOR, self.search_bar)
            search_input.clear()
            search_input.send_keys(keywords)
            search_input.send_keys(Keys.RETURN)

            # Extract total number of search results
            page_title_element = self.driver.find_element(By.CSS_SELECTOR, '#page-title')
            total_results_text = page_title_element.text
            total_results = int(total_results_text.split()[0])

            # Extract search results titles and links
            search_results = self.driver.find_elements(By.CSS_SELECTOR, '.highwire-cite-title')
            titles = [result.text for result in search_results]

            # Store total results and titles in a dictionary
            search_data = {'total_results': total_results, 'titles': titles}

            # Save search results to a JSON file
            with open('biorxiv_search_results.json', 'w') as f:
                json.dump(search_data, f)

    @app.command()
    def search_biorxiv(keywords: str):
        # Initialize Chrome WebDriver
        driver = webdriver.Chrome()

        # Initialize BioRxivScraper instance
        biorxiv_scraper = BioRxivScraper(driver)

        # Navigate to BioRxiv
        biorxiv_scraper.navigate()

        # Perform search
        biorxiv_scraper.search(keywords)

        # Close the WebDriver
        driver.quit()

    if __name__ == "__main__":
        app()


    ```

5. Save the file and exit the editor.

### Step 8: Running the Script
1. In the terminal, make sure your virtual environment is activated.
2. Run the Python script from the terminal:
   ```
   python scrap_biorxiv.py "dengue AND arbo"
   ```

This will execute the script, scrape BioRxiv for articles related to the specified keywords, and store the results in a JSON file named `biorxiv_search_results.json` in the current directory.
