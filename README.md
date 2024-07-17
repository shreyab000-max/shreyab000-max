import requests
from bs4 import BeautifulSoup
import pandas as pd

# URL of the website to scrape
base_url = 'https://hprera.nic.in/PublicDashboard'

# Send a GET request to the website
response = requests.get(base_url)
soup = BeautifulSoup(response.content, 'html.parser')

# Find the section containing the Registered Projects
registered_projects = soup.find_all('a', {'href': lambda L: L and L.startswith('https://hprera.nic.in/RealEsateProject')})

# Get the first 6 project links
project_links = [proj['href'] for proj in registered_projects[:6]]

# Lists to hold the project details
gstin_list = []
pan_list = []
name_list = []
address_list = []

# Loop through each project link to get the details
for link in project_links:
    project_response = requests.get(link)
    project_soup = BeautifulSoup(project_response.content, 'html.parser')
    
    gstin_no = project_soup.find(text='GSTIN No').find_next('span').text.strip()
    pan_no = project_soup.find(text='PAN No').find_next('span').text.strip()
    name = project_soup.find(text='Name').find_next('span').text.strip()
    address = project_soup.find(text='Permanent Address').find_next('span').text.strip()
    
    gstin_list.append(gstin_no)
    pan_list.append(pan_no)
    name_list.append(name)
    address_list.append(address)

# Create a DataFrame to organize the data
data = {
    'GSTIN No': gstin_list,
    'PAN No': pan_list,
    'Name': name_list,
    'Permanent Address': address_list
}
df = pd.DataFrame(data)

# Print the DataFrame
print(df)

# Save the DataFrame to a CSV file
df.to_csv('registered_projects.csv', index=False)
