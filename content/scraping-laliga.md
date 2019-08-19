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

```
['https://www.msn.com/en-us/sports/soccer/la-liga/scores/',

 'https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428',
 
 'https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428-d-20190331',
 
 'https://www.msn.com/en-us/sports/soccer/la-liga/scores/sp-d-20190428-d-20190331-d-20190303']
 ```
