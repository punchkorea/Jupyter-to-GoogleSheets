# Automate the collection of Naver News articles with keywords in Google Sheets using Python

> Please replace "daily-articles.json" file with your Google API credential file.

## :newspaper: Why scrape news articles? 

You may have a need for being updated about the latest news in your professional field or a specific topic of interest. The conventional way of opening a news website, typing your field of interest, scrolling through all news feeds, opening the same links again and again, and saving some useful links in separate files can be quite repetitive for daily usage. Thus, we attempted to automate this process using simple and open source tools such as Jupyter Notebook to run Python code and Google Sheets, where we can save all necessary data in a visually appealing way.

The contents of this readme file:
In Part One, we will share how to create Google API credentials in order to access Google Sheets without manually opening it.
In Part Two, we will discuss how to do web scraping in Python in Jupyter Notebook.
In Part Three, we will connect output data and Google Sheets in order to save the data for further analysis and displaying. 

## :page_facing_up: Part One: Google API credentials creation

1. Create a Google Account if you do not have one.
2. Open Google Developer Console.
3. Create a New Project by clicking on “Create Project” button,
Type your project name and ID (it cannot be changed after this stage) and click “Create”
4. Open your Project (refresh page if you do not see it) and click API -> Dashboard
5. In the APIs Dashboard, click “Enable APIs” to enable all required APIs.
New page is opened, where you can type “Google Drive API” and “Google Sheets API”.
Enable both APIs.
6. After enabling Google Drive API, go to the “Credentials” section and click “Create Credentials” -> “Service Account”. 
We need to create these credentials to use them to access Google Sheets from Jupyter Notebook. 
7. Start creating credentials. Type any name and skip (if you want) optional settings. 
For the role, you can select “Owner” to have full access (for your usage only).
8. After you create the credentials, you can see them in the “Service Accounts” sections. Now in order to access the key, go to the “Manage Keys” section.
9. Create a key for this service account. Select “json” as the type of the file. When you create a key, it will automatically download the key. 
In case it did not download automatically, go to the KEYS tab and download the key manually. The key is basically a .json format file which has all credential information of a service account. The content of the key is as in daily-articles.json file.

Well done! You have your Google API credentials ready and now let’s get into coding!

## :dart: Part Two: Web Scraping using Python
1. Install Jupyter Notebook to run [Python](https://jupyter.org/install) 
2. In order to get information from a webpage, we need to install web scraping libraries, and we used [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)  due to its easy usage. 
3. Run the following command from cmd/terminal.
```
pip install beautifulsoup4
```
4. Install Requests to access <html> code of the webpage. Run the following command from cmd/terminal.
```
pip install requests
```
5. Now when you have the environment installed, create a blank python file.
6. Import all our libraries. We will need numpy to manipulate the data. 
```
import requests
from bs4 import BeautifulSoup
import numpy as np
```
7. Define you keywords as an array, For example:
```
keywords = [“AI”, “neuralink”, “deep+learning”
```
8. Go to the desired website and type “EXAMPLE” in the search bar. Here you can apply all sorting needed, for example “Newest” or “Popular” etc. Copy the result page url. 
For example, you get the URL as: 
https://search.naver.com/…...s&query=EXAMPLE&sm=tab_….
In the URL, you can see your EXAMPLE word as equals to “query”. This is the place where you can put any other keyword and save the url for each keyword. We used for loop and concatenate the URL string
```
url_list = []
for n in np.arange(0, len(keywords)):
url_each = "https://m.search.naver.com/..query="+keywords[n]+"&..."
url_list.append(url_each)
```
9. Now when you have all URLs from which to get data, use the following commands to retrieve content of the page.
```
r = requests.get(url_each)
page = r.content
```
10. In order to understand which information you need from the content, go to the wanted page (the result page) and right click “View page source”. You will see "html" of the page (which is the content) and find the wanted type (html tag type) of the information. For example, in news websites, each news has a title in <a> tag with a link attached to it (href component of "anchor" tag). Now, use the following easy lines of code to get the wanted section.
```
soup = BeautifulSoup(page, 'html5lib')
news = soup.find_all('a', class_='news_title')
```
Please note that these parameters (tag type, class name) depend on the website you are scraping. 
11. You can manage the data as you want, but in our case, we saved data regarding each keyword as an object. Then, we combined data for all keywords, which resulted in a big array that will be used as an output dataframe.
12. Import Pandas Library to work with data frames. Save the resultant big array as a dataframe using the installed library. 
```
import pandas as pd
table =  pd.DataFrame(big_list)
```

Well done on preparing the data output! Now it is time to connect the Jupyter Notebook to Google Sheets.
	
## :dizzy: Part Three: Connecting Jupyter Notebook with Google Sheets
1. To login into Google Sheets using Jupyter Notebook, install the following libraries.
```
import gspread
from df2gspread import df2gspread as d2g
```
2. Get your Google API credentials and authorize into your Google Account.
```
from oauth2client.service_account import ServiceAccountCredentials
scope = ['https://spreadsheets.google.com/feeds',
         'https://www.googleapis.com/auth/drive']
credentials = ServiceAccountCredentials.from_json_keyfile_name('./YOUR CREDENTIALS FILE.json', scope) 
gc = gspread.authorize(credentials)
```
Important note: all your Python files should be in the same folder as your credentials key file (.json). If it shows an error at this stage, go to your Google Developer Console and check if both Google Drive and Google Sheets APIs are enabled in your project.
3. Create a google spreadsheet file and share it with the service account email you created before. To see the service account email, open the credentials key .json file and look for “client_email”.
4. Now you will upload your dataframe (big table) into the Google Sheets using the following command
```
d2g.upload(table, spreadsheet_key, s_name, credentials=credentials)
```
Let’s go one by one for each variable,
- table => dataframe you created in Step 12 of Part 2.
- spreadsheet_key => Open your google spreadsheet and look at the URL.
The spreadsheet_key is basically the highlighted part of the URL as shown below.
https://docs.google.com/spreadsheets/d/1..longstringofletters..w/edit..
So it is a string of letters and numbers which come after “/d/” and before “/edit..” parts.
- s_name => spreadsheet name, you can define as any string (for example, s_name = “TEST”)
- credentials =>  your Google API credentials, which you defined in Step 2 of Part 3.

You can define optional variables if you want to to change upload method, see more [here](https://df2gspread.readthedocs.io/en/latest/examples.html). 

If there are no errors, Jupyter displays the following message: 
```
<Worksheet ‘TEST’ id:123..example..123>
```
:white_check_mark: Great job!
Now go to the spreadsheet you created and you can see your “TEST” spreadsheet and the dataframe in it. 

> Finito~

Please see full post [on our Blog](https://punchkorea.com/automate-the-collection-of-naver-news-articles-with-keywords-in-google-sheets/).




