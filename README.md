# eurovision-2021-votes

## Source
Data source is the [Eurovision website](https://eurovision.tv/event/rotterdam-2021/grand-final/results/).

## Method

1. Files were downloaded with individual names `eurovision_html/<country>.html` (as `requests` scraping is blocked by Eurovision's website).

2. After downloading, the files were processed using the following code:

    ```py
    import requests
    from bs4 import BeautifulSoup
    import re
    import pandas as pd

    matches = r'(\d{1,2})(th|st|rd|nd)'

    countries = ['israel', 'albania', 'australia', 'austria', 'azerbaijan', 'belgium', 'bulgaria', 'croatia', 'cyprus', 'czech-republic', 'denmark', 'estonia', 'finland', 'france', 'georgia', 'germany', 'greece', 'iceland', 'ireland', 'italy', 'latvia', 'lithuania', 'malta', 'moldova', 'netherlands', 'north-macedonia', 'norway', 'poland', 'portugal', 'romania', 'russia', 'san-marino', 'serbia', 'slovenia', 'spain', 'sweden', 'switzerland', 'ukraine', 'united-kingdom']

    df = pd.DataFrame()

    for voting_country in countries:
        with open(f'./eurovision_html/{voting_country}.html') as f:
            html = f.read()
        soup = BeautifulSoup(html)

        table = soup.find_all('table', {'class': 'w-full'})[-1]

        rows = table.tbody.find_all('tr')

        for row in rows:
            country = row.find_all('td')[0].text.strip()
            
            text = row.find_all('td')[-1].text.strip()
            g = re.search(matches, text)
            if g.groups():
                televote_ranking = g.groups()[0]
            
            text = row.find_all('td')[-2].text.strip()
            g = re.search(matches, text)
            if g.groups():
                jury_ranking = g.groups()[0]
            
            individual_juror_votes = [x.text for x in row.find_all('td')[2:7]]
            
            d = {'country': country, 'televote_ranking': televote_ranking, 'jury_ranking': jury_ranking}
            for juror_num, vote in enumerate(individual_juror_votes):
                d[f'juror_{juror_num}'] = vote

            s = pd.Series(d, name=voting_country)

            df = df.append(s)

    df.index.name = 'voting_country'

    df.to_csv('votes.csv')

    df.to_excel('votes.xls')

    ```