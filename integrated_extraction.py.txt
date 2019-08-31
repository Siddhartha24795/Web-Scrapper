import requests
import bs4
import pandas as pd
arline='air-niugini'
rating,airlines=[],[]
type_of_traveller, cabin_flown, date_flown, seat_comfort, cabin_staff_service, food_beverages, value_for_money, recommended,name,country,content ,inflight_entertaintment=[], [], [], [], [], [], [], [], [], [], [], []
for page_no in range(1,2):
    #url='https://www.airlinequality.com/airline-reviews/aero-vip/'
    url='https://www.airlinequality.com/airline-reviews/'+arline+'/page/'+str(page_no)+'/?sortby=post_date%3ADesc&pagesize=100'
    res = requests.get(url)
    soup = bs4.BeautifulSoup(res.text, 'lxml')
    articles=soup.find_all('article',itemprop='review')
    for art in articles:
        dict = {'tot': 'none', 'cf': 'none', 'df': 'none', 'sc': 'none', 'cs': 'none', 'fb': 'none', 'vfm': 'none',
                'rc': 'none', 'name': 'none', 'ctry': 'none', 'cntnt': 'none', 'ife': 'none','rate':'none'}
        ovr_rte=art.find_all('div',itemprop='reviewRating') or art.find_all('div',class_='rating-10')
        dict['rate']=ovr_rte[0].get_text().strip()
        s = art.h3.get_text()
        if '(' in s:
            f_indx = s.index('(')
            l_indx = s.index(')')
            nm = s[:f_indx]
            nm = nm.replace('\n\n', '')
            dict['name'] = nm
            dict['ctry'] = s[f_indx + 1:l_indx]
        else:
            dict['name']=s.strip()
            dict['ctry']='none'
        c = art.find_all('div', class_='text_content')
        dict['cntnt'] = c[0].get_text()
        for y in art.find_all('tr'):
            col = (y.td.string).strip()
            # print(col, end=' ')
            stars = y.find_all('span', class_='star fill')
            if len(stars) != 0:
                # print( len(stars))
                if col == 'Seat Comfort':
                    dict['sc'] = len(stars)
                if col == 'Cabin Staff Service':
                    dict['cs'] = len(stars)
                if col == 'Food & Beverages':
                    dict['fb'] = len(stars)
                if col == 'Value For Money':
                    dict['vfm'] = len(stars)
                if col == 'Inflight Entertainment':
                    dict['ife'] = len(stars)

            else:
                for u in y.td.find_next_sibling("td"):
                    # print( u.string)
                    if col == 'Type Of Traveller':
                        dict['tot'] = u.string
                    if col == 'Cabin Flown':
                        dict['cf'] = u.string
                    if col == 'Date Flown':
                        dict['df'] = u.string
                    if col == 'Recommended':
                        dict['rc'] = u.string

        type_of_traveller.append(dict['tot'])
        cabin_flown.append(dict['cf'])
        date_flown.append(dict['df'])
        seat_comfort.append(dict['sc'])
        cabin_staff_service.append(dict['cs'])
        food_beverages.append(dict['fb'])
        value_for_money.append(dict['vfm'])
        recommended.append(dict['rc'])
        name.append(dict['name'])
        country.append(dict['ctry'])
        content.append(dict['cntnt'])
        inflight_entertaintment.append(dict['ife'])
        rating.append(dict['rate'])
        airlines.append(arline)
columns = {'airlines':airlines,'author':name,'author_country':country,'content':content,'type_of_traveller': type_of_traveller, 'cabin_flown': cabin_flown, 'date_flown': date_flown,
           'seat_comfort': seat_comfort, 'cabin_staff_service': cabin_staff_service, 'food_beverages': food_beverages,
           'value_for_money': value_for_money, 'recommended': recommended,'inflight_entertaintment':inflight_entertaintment,'overall_rating':rating}

df = pd.DataFrame(columns)
writer=pd.ExcelWriter(arline+'.xlsx',engine='xlsxwriter')
df.to_excel(writer,sheet_name='Sheet1')
writer.save()
print('Successfully Done!!!!',arline)

