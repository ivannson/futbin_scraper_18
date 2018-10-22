
# coding: utf-8

import requests
import bs4
import pandas as pd
from progressbar import ProgressBar
from datetime import datetime, timedelta
import re


def get_stat(html_target, i, is_number, name):
    stat_name = player.select(html_target)
    if i == 0:
        k = i
        if is_number == True:
            stat_value = float(stat_name[k].getText(strip=True))
        if is_number == False:
            stat_value = stat_name[k].getText(strip=True)
    else:
        for k in range(i):
            if is_number == True:
                stat_value = float(stat_name[k].getText(strip=True))
            if is_number == False:
                stat_value = stat_name[k].getText(strip=True)
    stat_name_list.append(name)
    stat_values_list.append(stat_value)
    return(stat_value)

def add_to_list(name, values):
    stat_name_list.append(name)
    stat_values_list.append(values)
    
def isfloat(value):
  try:
    float(value)
    return True
  except ValueError:
    return False


#obtaining traits and specialities list
traits_list = []
spec_list = []
list_of_stats = []
stat_values_list = []
stat_name_list = []
date_added = []

url1 = 'http://www.futbin.com/18/players'
res1 = requests.get(url1)
res1.raise_for_status()
player1 = bs4.BeautifulSoup(res1.text, "html.parser")
for i in range(31):
    traits = player1.select('ul li ul li ul ul li span')[i].getText(strip=True)
    traits_list.append(traits)
  
for i in range(31, 47, 1):
    spec = player1.select('div div div ul li ul li ul.dropdown-menu li.tands_li.notxtselect span')[i].getText(strip=True)
    spec_list.append(spec)
  

pbar = ProgressBar()
date_added = []
q = 1

for n in pbar(range(q,20001)):
    
   
    url = 'http://www.futbin.com/18/player/' + str(n)
    try:
        res = requests.get(url)
        res.raise_for_status()
    except requests.exceptions.HTTPError:
        date_added.append(None)
        continue
    player = bs4.BeautifulSoup(res.text, "html.parser")

    
    number_of_stats = 35
    
    spec_list[13] = 'Strength Spec'
#---------- 

    #get player position
    position = get_stat('.pcdisplay-pos', 0, False, "Position")
    if position == 'GK':
        date_added.append(None)
        continue  
        
#----------  

    #get player rating
    rating = get_stat('.pcdisplay-rat', 0, True, "Rating")
    
#---------- 

    #get player ingame stats and names
    stat_values = player.select('.stat_val .stat_val')
    for i in range(number_of_stats):
        d = isfloat(stat_values[i].getText(strip=True))
        if d is True: 
            stat_values_float = float(stat_values[i].getText(strip=True))
            stat_values_list.append(stat_values_float)
        if d is False:
            stat_values_float = 0.0
            stat_values_list.append(stat_values_float)
    
    stat_names = player.select('.ig-stat-name-tooltip')
    for i in range(number_of_stats):
        stat_names1 = stat_names[i].getText(strip=True)
        stat_name_list.append(stat_names1)
        
 #----------         

    #get some other stats
    info_stats_values = []
    info_stats_names = []

    for i in range(15):

        info_names = player.select('#info_content th')[i]
        info_stats_names.append(info_names.getText(strip=True))
        info_stats = player.select('#info_content .table-row-text')[i]
        info_stats_values.append(info_stats.getText(strip=True))   
        
        if i in [4, 5, 6, 9]:
            x = isfloat(info_stats_values[i])
            if x is False:
                info_stats_values.insert(i, 0.0)
                info_stats_names.insert(i, 'Intl. Rep')
                   
        if i == 8:
            info_stats_values[i] = float(info_stats_values[i][:3])
        
        if i == 13: 
            date_added.append(info_stats_values[i])
        if i != 13:    
            stat_name_list.append(info_stats_names[i])
            stat_values_list.append(info_stats_values[i])
        
        
            
 #----------  
    #determine card version
    version = player.select('#version0')[0].get('data-version')
    add_to_list("Version", version)
    
 #----------

    #get number of games played on PS4
    games_played_ps4 = player.select(' .ps4-pgp-data')[5]
        
    games_played_ps4_number = games_played_ps4.getText(strip=True).replace(',','')
    y = isfloat(games_played_ps4_number)
    if y is False:
        games_played_ps4_float = 0.0
        add_to_list("PS4 Games Played", games_played_ps4_float)
    if y is True:
        games_played_ps4_float = float(games_played_ps4_number)
        add_to_list("PS4 Games Played", games_played_ps4_float)
    
 #---------- 
    
    #how many days has the card been in the game 
    #print(n, q)
    date_added_f = datetime.strptime(date_added[n-q], '%Y-%m-%d')
    date_today = datetime.strptime('2018-09-12', '%Y-%m-%d') #fixed date because game is dead
    days_in_game = float((date_today - date_added_f).days)
    add_to_list("Days in game", days_in_game)
    
 #----------
    #get traits and specialities

    specialities = player.select('#specialities_content')
    specialities1 = specialities[0].getText(strip=True)
    for i in range(16):
        has_spec = float(spec_list[i] in specialities1)
        stat_name_list.append(spec_list[i])
        stat_values_list.append(has_spec)

    traits = player.select('#traits_content')
    traits1 = traits[0].getText(strip=True)
    for i in range(31):
        has_trait = float(traits_list[i] in traits1)
        stat_name_list.append(traits_list[i])
        stat_values_list.append(has_trait)
        
 #----------

    #normalising data: number of games per day
    games_day = games_played_ps4_float/days_in_game

 #----------   

    #create a list of dictionaries
    for i in range(number_of_stats):
        stats = dict(zip(stat_name_list, stat_values_list))
    list_of_stats.append(stats)    
    
         
    table = pd.DataFrame(list_of_stats, columns = stat_name_list[:101])

    table.to_csv('/Path/to/output.csv')
table.head()

