title: Scraping LaLiga Statistics 2018/19
date: 08/19/2019
author: Parth Shah

## Environment Setup
I have used Python 3.6 for this tutorial. The required libraries are as follows:

- numpy
- pandas
- bs4 (BeautifulSoup)
- urllib (Module for working with URLs)

## Web Scraping from msn.com

![image](images/laliga-home.png)

The website lists the scores for each month on a separate page and it follows this pattern: 

*https://www.msn.com/en-us/sports/soccer/la-liga/scores/*

*https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428*

*https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428-d-20190331*

*https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428-d-20190331-d-20190303*

As we can see notice the pattern here:(*I*)

http://............./sp-d-(*new-match-date*)-d-(*past-date*)-d-(*past-date*).

I recorded those dates in a list and created a function that gives me all the links.

```python
links = ['20190428', '20190331', '20190303', '20190203', '20190106', '20181209', '20181111', '20181014', '20180916', '20180819']

def fetch_website(links):
    urls = []
    url = 'https://www.msn.com/en-us/sports/soccer/la-liga/scores/'
    urls.append(url)
    url += 'sp'
    for i, l in  enumerate(links):
        url += '-d-' + links[i]
        urls.append(url)
    return urls
    
urls = fetch_website(links)
```

`['https://www.msn.com/en-us/sports/soccer/la-liga/scores/',
 'https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428',
 'https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428-d-20190331',
 'https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428-d-20190331-d-20190303'...]`
 
 Once we have the links, all we have to do is create a script that runs through all the links and gives us the desired output. The page source of the web page looks like this:

![image](images/laliga-pagesource.jpg)

Our scores are listed in the *sectionsgroup* container. Each team scores are in a *tbody* tag distributed by *even rowlink* and *odd rownlink* classes. Now we can use the power of BeautifulSoup to scrape the contents with just two functions:

- *find*: This function takes the name of the tag as string input and returns the first found match from the webpage. 
	- Here we will find the *div* container with class *sectionsgroup* that contains all the score results.
- *findAll*: to extract all the occurrences of a particular tag from the page.
	- Here we will *findAll* the *tbody* tags with classes *even rowlink* & *odd rowlink* and then loop through each of them to get the contents.

The following is the code to scrape just one webpage:
```python
url = 'https://www.msn.com/en-us/sports/soccer/la-liga/scores/'
html = urllib.request.urlopen(url).read()
raw = bs4.BeautifulSoup(html, 'lxml')
results = raw.find('div', {'class': 'sectionsgroup'}).findAll('tbody', {'class': ['even rowlink', 'odd rowlink']})
for r in results:
	home_team = r.find('td', {'class': 'teamname teaminline alignright size23'}).text.strip()
	away_team = r.find('td', {'class': 'teamname size23'}).text.strip()
	match_date = r.find('td', {'class': 'groupingcolumn paddingleft'}).find('div', {'class': 'matchdate'}).text.strip()
	scores = r.findAll('td', {'class': 'teamscore'})

	home_team_score = scores[0].text.strip()
	away_team_score = scores[1].text.strip()

	data_link = 'https://www.msn.com' + r.find('a').get('href')
```

This gives us the team names, match date, their scores and the *data_link*(link that takes us to a separate page for detailed statistics of each match) using just *find* and *findAll* tags. Now to run through all pages we just have to run this for each and every link and append the results to a list.

```python
def get_html(urls):
    teams = []
    for url in urls:
        html = urllib.request.urlopen(url).read()
        raw = bs4.BeautifulSoup(html, 'lxml')
        results = raw.find('div', {'class': 'sectionsgroup'}).findAll('tbody', {'class': ['even rowlink', 'odd rowlink']})
        for r in results:
            home_team = r.find('td', {'class': 'teamname teaminline alignright size23'}).text.strip()
            away_team = r.find('td', {'class': 'teamname size23'}).text.strip()
            match_date = r.find('td', {'class': 'groupingcolumn paddingleft'}).find('div', {'class': 'matchdate'}).text.strip()
            scores = r.findAll('td', {'class': 'teamscore'})
            
            home_team_score = scores[0].text.strip()
            away_team_score = scores[1].text.strip()

            data_link = 'https://www.msn.com' + r.find('a').get('href')
            
			teams.append([match_date.split(',')[0], match_date.split(',')[1][1:], home_team, away_team,
                          home_team_score, away_team_score, data_link])
		time.sleep(5)
	return teams
```

This gives us results for each match in a list that can be easily converted into a dataframe. 

`
[['Sun',
  'May 19',
  'Eibar',
  'Barça',
  '2',
  '2',
  'https://www.msn.com/en-us/sports/soccer/la-liga/eibar-v-barcelona/game-center/sp-id-80402000001009693'],
 ['Sun',
  'May 19',
  'Real Madrid',
  'Betis',
  '0',
  '2',
  'https://www.msn.com/en-us/sports/soccer/la-liga/real-madrid-v-real-betis/game-center/sp-id-80402000001009694'],
  ...
 ]
`
 
