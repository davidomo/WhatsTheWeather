
# What is the weather like as you approach the equator?

### A data analysis showing the relationship of weather patterns to the equator

##### Notes and Considerations:

- The data used in this example was pulled at 11:00 AM EST on December 23, 2017.
- Graphs in this output are flipped along the axes, that is, the independent is on the Y-axis and the dependent is on the X-axis. I purposefully flipped in this case since we are looking at the relationship of how far away a city is from the equator to its current weather. On the graphs, the line y = 0 would represent the equator, making the graphs an easier representation of how we generally look at the world.
- The data pulled through OpenWeatherMap.com is in imperial units, meaning temp is displayed in Fahrenheit.
- Feel free to run this exercise at different times to see how time and data change the data.

Observations:

- In this data set, the most extreme temperatures do not happen exactly at the equator or at the northern/southern poles. The highest temperature happens near 10.274128 degrees North at the town of Am Timan, Chad, near the Saharan Desert. The coldest temperature happens near 62.42979 degrees North at the town of Kangalassy, Russia, along the Lena River.
- In this data set, even though different equators are in different "seasons", Humidity, Wind Speed, and Cloud Coverage were normalized between the two equators. This means that, besides a current temperature, an aggregate of the two hemispheres are rather close in the other three categories.
- The effect of the different seasons on the data set is apparent in the Temperature scatter plot. Generally, the locations found below the equator are warmer than those above. 
- While this data analysis answers the question it set out to answer, the data doesn't take into account different physiological qualities of the randomly generated coordinates. I believe the next step to answer this question is to look at other qualities such as elevation, percentage of land that is natural v urbanized, or even a look at historical data to see how the land surrounding the area effects the temperature. An example to explore would be to look at places that are near mountain ranges with a higher historical rate of rainfall per year and see if that keeps temperature down even if the location may be close to the equator. However, generally speaking, you can confirm that temperature rises as a city is closer to the equator.


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import requests as req
from citipy import citipy
import time
import json
import time
```


```python
# Run to get a random set of longitude and latitude coordinates
list1 = np.random.uniform(-90,90, size = 1800)
list2 = np.random.uniform(-180,180, size = 1800)
```


```python
#Create a df
list1_df = pd.DataFrame({"Lat": list1, "Lng": list2})
list1_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Lat</th>
      <th>Lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-8.425875</td>
      <td>-113.445745</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-7.067614</td>
      <td>175.715527</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-70.629471</td>
      <td>-25.455300</td>
    </tr>
    <tr>
      <th>3</th>
      <td>50.397708</td>
      <td>114.131234</td>
    </tr>
    <tr>
      <th>4</th>
      <td>41.792746</td>
      <td>177.089244</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Make new columns in data frame to fill in with city data
list1_df["City"] = ""
list1_df["Country"] = ""

# Use citipy to find the nearest city to given random coords and write into the DF
for index, row in list1_df.iterrows():
    city = citipy.nearest_city(row["Lat"], row["Lng"])
    list1_df.set_value(index, "City", city.city_name)
    list1_df.set_value(index, "Country", city.country_code)

# Remove duplicate cities
list1_df = list1_df.drop_duplicates(["City"], keep='first')

# Count unique "Cities" to ensure a significant number for later manipulation. In this instance, 500 is the desired minimum.
# However, a city may not have any weather data in the next step, so we are going to shoot for 650 unique cities.
# On first run through, I started with a sample of 700 randomized coordinates. This lent only 352 unique cities.
# Attempt two: 900 randomized coords, 445 unique cities.
# Attempt three: 1100 randomized coords, 485 unique cities.
# Attempt four: 1200 randomized coords, 546 unique cities.
# Attempt five: 1800 randomized coords, 694 unique cities.

list1_df.count()
```




    Lat        694
    Lng        694
    City       694
    Country    694
    dtype: int64




```python
# Build the API URL
akey = "34ae2627193903bb86906bf116b20b46"
url = "http://api.openweathermap.org/data/2.5/weather?q="

list1_df["Temp"] = ""
list1_df["Hum"] = ""
list1_df["Cloud"] = ""
list1_df["Wind"] = ""

SleepCounter = 0
PullCounter = 0
BatchCounter = 1

# Loop through the APIs to construct new columns in df
for index, row in list1_df.iterrows():
    try:
        query = url + row["City"].replace(" ","+") + "," + row["Country"] + "&appid=" + akey + "&units=imperial"
        get = req.get(query)
        getJ = get.json()
        list1_df.set_value(index, "Temp", getJ["main"]["temp"])
        list1_df.set_value(index, "Hum", getJ["main"]["humidity"])
        list1_df.set_value(index, "Cloud", getJ["clouds"]["all"])
        list1_df.set_value(index, "Wind", getJ["wind"]["speed"])
    except:
        list1_df.set_value(index, "Temp", "FAIL")
    
    PullCounter += 1
    
    SleepCounter += 1
    
    # If loop to ensure not overloading the weather API
    if SleepCounter == 40:
        print("~~~ Break Time ~~~")
        time.sleep(10)
        print("")
        SleepCounter = 0
        BatchCounter += 1
    
    # Printing API link
    print("Processing Record " + str(PullCounter) + " of Set " + str(BatchCounter) +" | " + row["City"])
    print(query)
```

    Processing Record 1 of Set 1 | puerto ayora
    http://api.openweathermap.org/data/2.5/weather?q=puerto+ayora,ec&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 2 of Set 1 | lolua
    http://api.openweathermap.org/data/2.5/weather?q=lolua,tv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 3 of Set 1 | ushuaia
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 4 of Set 1 | duldurga
    http://api.openweathermap.org/data/2.5/weather?q=duldurga,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 5 of Set 1 | nikolskoye
    http://api.openweathermap.org/data/2.5/weather?q=nikolskoye,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 6 of Set 1 | bistret
    http://api.openweathermap.org/data/2.5/weather?q=bistret,ro&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 7 of Set 1 | klaksvik
    http://api.openweathermap.org/data/2.5/weather?q=klaksvik,fo&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 8 of Set 1 | zyryanka
    http://api.openweathermap.org/data/2.5/weather?q=zyryanka,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 9 of Set 1 | northam
    http://api.openweathermap.org/data/2.5/weather?q=northam,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 10 of Set 1 | rawson
    http://api.openweathermap.org/data/2.5/weather?q=rawson,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 11 of Set 1 | rikitea
    http://api.openweathermap.org/data/2.5/weather?q=rikitea,pf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 12 of Set 1 | olga
    http://api.openweathermap.org/data/2.5/weather?q=olga,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 13 of Set 1 | harrogate
    http://api.openweathermap.org/data/2.5/weather?q=harrogate,gb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 14 of Set 1 | codrington
    http://api.openweathermap.org/data/2.5/weather?q=codrington,ag&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 15 of Set 1 | bundaberg
    http://api.openweathermap.org/data/2.5/weather?q=bundaberg,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 16 of Set 1 | saint-joseph
    http://api.openweathermap.org/data/2.5/weather?q=saint-joseph,re&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 17 of Set 1 | taolanaro
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro,mg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 18 of Set 1 | arraial do cabo
    http://api.openweathermap.org/data/2.5/weather?q=arraial+do+cabo,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 19 of Set 1 | airai
    http://api.openweathermap.org/data/2.5/weather?q=airai,pw&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 20 of Set 1 | puerto escondido
    http://api.openweathermap.org/data/2.5/weather?q=puerto+escondido,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 21 of Set 1 | athabasca
    http://api.openweathermap.org/data/2.5/weather?q=athabasca,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 22 of Set 1 | tasiilaq
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 23 of Set 1 | ukiah
    http://api.openweathermap.org/data/2.5/weather?q=ukiah,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 24 of Set 1 | yola
    http://api.openweathermap.org/data/2.5/weather?q=yola,ng&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 25 of Set 1 | tafresh
    http://api.openweathermap.org/data/2.5/weather?q=tafresh,ir&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 26 of Set 1 | port alfred
    http://api.openweathermap.org/data/2.5/weather?q=port+alfred,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 27 of Set 1 | bermeo
    http://api.openweathermap.org/data/2.5/weather?q=bermeo,es&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 28 of Set 1 | yellowknife
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 29 of Set 1 | provideniya
    http://api.openweathermap.org/data/2.5/weather?q=provideniya,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 30 of Set 1 | saskylakh
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 31 of Set 1 | hithadhoo
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo,mv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 32 of Set 1 | pisco
    http://api.openweathermap.org/data/2.5/weather?q=pisco,pe&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 33 of Set 1 | lebu
    http://api.openweathermap.org/data/2.5/weather?q=lebu,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 34 of Set 1 | vaini
    http://api.openweathermap.org/data/2.5/weather?q=vaini,to&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 35 of Set 1 | hobart
    http://api.openweathermap.org/data/2.5/weather?q=hobart,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 36 of Set 1 | mataura
    http://api.openweathermap.org/data/2.5/weather?q=mataura,pf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 37 of Set 1 | ponta do sol
    http://api.openweathermap.org/data/2.5/weather?q=ponta+do+sol,cv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 38 of Set 1 | bethel
    http://api.openweathermap.org/data/2.5/weather?q=bethel,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 39 of Set 1 | shimoda
    http://api.openweathermap.org/data/2.5/weather?q=shimoda,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 40 of Set 2 | homer
    http://api.openweathermap.org/data/2.5/weather?q=homer,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 41 of Set 2 | dingle
    http://api.openweathermap.org/data/2.5/weather?q=dingle,ie&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 42 of Set 2 | khonuu
    http://api.openweathermap.org/data/2.5/weather?q=khonuu,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 43 of Set 2 | fabiansebestyen
    http://api.openweathermap.org/data/2.5/weather?q=fabiansebestyen,hu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 44 of Set 2 | bluff
    http://api.openweathermap.org/data/2.5/weather?q=bluff,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 45 of Set 2 | bilma
    http://api.openweathermap.org/data/2.5/weather?q=bilma,ne&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 46 of Set 2 | marsh harbour
    http://api.openweathermap.org/data/2.5/weather?q=marsh+harbour,bs&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 47 of Set 2 | jamestown
    http://api.openweathermap.org/data/2.5/weather?q=jamestown,sh&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 48 of Set 2 | iqaluit
    http://api.openweathermap.org/data/2.5/weather?q=iqaluit,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 49 of Set 2 | tsihombe
    http://api.openweathermap.org/data/2.5/weather?q=tsihombe,mg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 50 of Set 2 | bathsheba
    http://api.openweathermap.org/data/2.5/weather?q=bathsheba,bb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 51 of Set 2 | mahebourg
    http://api.openweathermap.org/data/2.5/weather?q=mahebourg,mu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 52 of Set 2 | sentyabrskiy
    http://api.openweathermap.org/data/2.5/weather?q=sentyabrskiy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 53 of Set 2 | atuona
    http://api.openweathermap.org/data/2.5/weather?q=atuona,pf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 54 of Set 2 | mogadishu
    http://api.openweathermap.org/data/2.5/weather?q=mogadishu,so&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 55 of Set 2 | cabo san lucas
    http://api.openweathermap.org/data/2.5/weather?q=cabo+san+lucas,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 56 of Set 2 | kaitangata
    http://api.openweathermap.org/data/2.5/weather?q=kaitangata,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 57 of Set 2 | mys shmidta
    http://api.openweathermap.org/data/2.5/weather?q=mys+shmidta,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 58 of Set 2 | riyadh
    http://api.openweathermap.org/data/2.5/weather?q=riyadh,sa&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 59 of Set 2 | merauke
    http://api.openweathermap.org/data/2.5/weather?q=merauke,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 60 of Set 2 | iquique
    http://api.openweathermap.org/data/2.5/weather?q=iquique,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 61 of Set 2 | ngawen
    http://api.openweathermap.org/data/2.5/weather?q=ngawen,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 62 of Set 2 | ribeira grande
    http://api.openweathermap.org/data/2.5/weather?q=ribeira+grande,pt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 63 of Set 2 | ambanja
    http://api.openweathermap.org/data/2.5/weather?q=ambanja,mg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 64 of Set 2 | te anau
    http://api.openweathermap.org/data/2.5/weather?q=te+anau,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 65 of Set 2 | bambous virieux
    http://api.openweathermap.org/data/2.5/weather?q=bambous+virieux,mu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 66 of Set 2 | lata
    http://api.openweathermap.org/data/2.5/weather?q=lata,sb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 67 of Set 2 | mar del plata
    http://api.openweathermap.org/data/2.5/weather?q=mar+del+plata,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 68 of Set 2 | palabuhanratu
    http://api.openweathermap.org/data/2.5/weather?q=palabuhanratu,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 69 of Set 2 | hasaki
    http://api.openweathermap.org/data/2.5/weather?q=hasaki,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 70 of Set 2 | illoqqortoormiut
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 71 of Set 2 | castro
    http://api.openweathermap.org/data/2.5/weather?q=castro,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 72 of Set 2 | thunder bay
    http://api.openweathermap.org/data/2.5/weather?q=thunder+bay,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 73 of Set 2 | barrow
    http://api.openweathermap.org/data/2.5/weather?q=barrow,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 74 of Set 2 | bredasdorp
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 75 of Set 2 | albury
    http://api.openweathermap.org/data/2.5/weather?q=albury,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 76 of Set 2 | katsuura
    http://api.openweathermap.org/data/2.5/weather?q=katsuura,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 77 of Set 2 | gaoua
    http://api.openweathermap.org/data/2.5/weather?q=gaoua,bf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 78 of Set 2 | longyearbyen
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen,sj&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 79 of Set 2 | touros
    http://api.openweathermap.org/data/2.5/weather?q=touros,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 80 of Set 3 | hermanus
    http://api.openweathermap.org/data/2.5/weather?q=hermanus,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 81 of Set 3 | tula
    http://api.openweathermap.org/data/2.5/weather?q=tula,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 82 of Set 3 | chato
    http://api.openweathermap.org/data/2.5/weather?q=chato,tz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 83 of Set 3 | avera
    http://api.openweathermap.org/data/2.5/weather?q=avera,pf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 84 of Set 3 | kapaa
    http://api.openweathermap.org/data/2.5/weather?q=kapaa,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 85 of Set 3 | umzimvubu
    http://api.openweathermap.org/data/2.5/weather?q=umzimvubu,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 86 of Set 3 | roald
    http://api.openweathermap.org/data/2.5/weather?q=roald,no&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 87 of Set 3 | pucara
    http://api.openweathermap.org/data/2.5/weather?q=pucara,pe&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 88 of Set 3 | davila
    http://api.openweathermap.org/data/2.5/weather?q=davila,ph&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 89 of Set 3 | busselton
    http://api.openweathermap.org/data/2.5/weather?q=busselton,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 90 of Set 3 | east london
    http://api.openweathermap.org/data/2.5/weather?q=east+london,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 91 of Set 3 | qasigiannguit
    http://api.openweathermap.org/data/2.5/weather?q=qasigiannguit,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 92 of Set 3 | hukuntsi
    http://api.openweathermap.org/data/2.5/weather?q=hukuntsi,bw&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 93 of Set 3 | saldanha
    http://api.openweathermap.org/data/2.5/weather?q=saldanha,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 94 of Set 3 | el balyana
    http://api.openweathermap.org/data/2.5/weather?q=el+balyana,eg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 95 of Set 3 | bur gabo
    http://api.openweathermap.org/data/2.5/weather?q=bur+gabo,so&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 96 of Set 3 | punta arenas
    http://api.openweathermap.org/data/2.5/weather?q=punta+arenas,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 97 of Set 3 | tignere
    http://api.openweathermap.org/data/2.5/weather?q=tignere,cm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 98 of Set 3 | hwange
    http://api.openweathermap.org/data/2.5/weather?q=hwange,zw&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 99 of Set 3 | kholodnyy
    http://api.openweathermap.org/data/2.5/weather?q=kholodnyy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 100 of Set 3 | faanui
    http://api.openweathermap.org/data/2.5/weather?q=faanui,pf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 101 of Set 3 | dikson
    http://api.openweathermap.org/data/2.5/weather?q=dikson,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 102 of Set 3 | guerrero negro
    http://api.openweathermap.org/data/2.5/weather?q=guerrero+negro,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 103 of Set 3 | dogondoutchi
    http://api.openweathermap.org/data/2.5/weather?q=dogondoutchi,ne&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 104 of Set 3 | fortuna
    http://api.openweathermap.org/data/2.5/weather?q=fortuna,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 105 of Set 3 | grindavik
    http://api.openweathermap.org/data/2.5/weather?q=grindavik,is&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 106 of Set 3 | tezu
    http://api.openweathermap.org/data/2.5/weather?q=tezu,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 107 of Set 3 | samusu
    http://api.openweathermap.org/data/2.5/weather?q=samusu,ws&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 108 of Set 3 | armidale
    http://api.openweathermap.org/data/2.5/weather?q=armidale,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 109 of Set 3 | balikpapan
    http://api.openweathermap.org/data/2.5/weather?q=balikpapan,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 110 of Set 3 | victoria
    http://api.openweathermap.org/data/2.5/weather?q=victoria,sc&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 111 of Set 3 | karkaralinsk
    http://api.openweathermap.org/data/2.5/weather?q=karkaralinsk,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 112 of Set 3 | tuatapere
    http://api.openweathermap.org/data/2.5/weather?q=tuatapere,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 113 of Set 3 | kangaatsiaq
    http://api.openweathermap.org/data/2.5/weather?q=kangaatsiaq,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 114 of Set 3 | srednekolymsk
    http://api.openweathermap.org/data/2.5/weather?q=srednekolymsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 115 of Set 3 | komsomolskiy
    http://api.openweathermap.org/data/2.5/weather?q=komsomolskiy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 116 of Set 3 | viedma
    http://api.openweathermap.org/data/2.5/weather?q=viedma,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 117 of Set 3 | tumannyy
    http://api.openweathermap.org/data/2.5/weather?q=tumannyy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 118 of Set 3 | esperance
    http://api.openweathermap.org/data/2.5/weather?q=esperance,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 119 of Set 3 | gizo
    http://api.openweathermap.org/data/2.5/weather?q=gizo,sb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 120 of Set 4 | avarua
    http://api.openweathermap.org/data/2.5/weather?q=avarua,ck&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 121 of Set 4 | gat
    http://api.openweathermap.org/data/2.5/weather?q=gat,ly&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 122 of Set 4 | carnarvon
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 123 of Set 4 | tasbuget
    http://api.openweathermap.org/data/2.5/weather?q=tasbuget,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 124 of Set 4 | toliary
    http://api.openweathermap.org/data/2.5/weather?q=toliary,mg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 125 of Set 4 | kastamonu
    http://api.openweathermap.org/data/2.5/weather?q=kastamonu,tr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 126 of Set 4 | buala
    http://api.openweathermap.org/data/2.5/weather?q=buala,sb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 127 of Set 4 | concarneau
    http://api.openweathermap.org/data/2.5/weather?q=concarneau,fr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 128 of Set 4 | kodiak
    http://api.openweathermap.org/data/2.5/weather?q=kodiak,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 129 of Set 4 | albany
    http://api.openweathermap.org/data/2.5/weather?q=albany,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 130 of Set 4 | bengkulu
    http://api.openweathermap.org/data/2.5/weather?q=bengkulu,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 131 of Set 4 | palauig
    http://api.openweathermap.org/data/2.5/weather?q=palauig,ph&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 132 of Set 4 | vladimiro-aleksandrovskoye
    http://api.openweathermap.org/data/2.5/weather?q=vladimiro-aleksandrovskoye,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 133 of Set 4 | lydenburg
    http://api.openweathermap.org/data/2.5/weather?q=lydenburg,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 134 of Set 4 | belushya guba
    http://api.openweathermap.org/data/2.5/weather?q=belushya+guba,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 135 of Set 4 | ironton
    http://api.openweathermap.org/data/2.5/weather?q=ironton,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 136 of Set 4 | havoysund
    http://api.openweathermap.org/data/2.5/weather?q=havoysund,no&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 137 of Set 4 | hilo
    http://api.openweathermap.org/data/2.5/weather?q=hilo,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 138 of Set 4 | makhachkala
    http://api.openweathermap.org/data/2.5/weather?q=makhachkala,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 139 of Set 4 | turtkul
    http://api.openweathermap.org/data/2.5/weather?q=turtkul,uz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 140 of Set 4 | ilulissat
    http://api.openweathermap.org/data/2.5/weather?q=ilulissat,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 141 of Set 4 | jiwani
    http://api.openweathermap.org/data/2.5/weather?q=jiwani,pk&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 142 of Set 4 | nanortalik
    http://api.openweathermap.org/data/2.5/weather?q=nanortalik,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 143 of Set 4 | cururupu
    http://api.openweathermap.org/data/2.5/weather?q=cururupu,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 144 of Set 4 | kargil
    http://api.openweathermap.org/data/2.5/weather?q=kargil,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 145 of Set 4 | namibe
    http://api.openweathermap.org/data/2.5/weather?q=namibe,ao&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 146 of Set 4 | dennery
    http://api.openweathermap.org/data/2.5/weather?q=dennery,lc&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 147 of Set 4 | san patricio
    http://api.openweathermap.org/data/2.5/weather?q=san+patricio,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 148 of Set 4 | pevek
    http://api.openweathermap.org/data/2.5/weather?q=pevek,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 149 of Set 4 | belaya gora
    http://api.openweathermap.org/data/2.5/weather?q=belaya+gora,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 150 of Set 4 | kavaratti
    http://api.openweathermap.org/data/2.5/weather?q=kavaratti,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 151 of Set 4 | segovia
    http://api.openweathermap.org/data/2.5/weather?q=segovia,co&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 152 of Set 4 | barentsburg
    http://api.openweathermap.org/data/2.5/weather?q=barentsburg,sj&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 153 of Set 4 | pangnirtung
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 154 of Set 4 | pangody
    http://api.openweathermap.org/data/2.5/weather?q=pangody,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 155 of Set 4 | eenhana
    http://api.openweathermap.org/data/2.5/weather?q=eenhana,na&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 156 of Set 4 | abu dhabi
    http://api.openweathermap.org/data/2.5/weather?q=abu+dhabi,ae&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 157 of Set 4 | gilgit
    http://api.openweathermap.org/data/2.5/weather?q=gilgit,pk&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 158 of Set 4 | palmer
    http://api.openweathermap.org/data/2.5/weather?q=palmer,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 159 of Set 4 | attawapiskat
    http://api.openweathermap.org/data/2.5/weather?q=attawapiskat,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 160 of Set 5 | tiznit
    http://api.openweathermap.org/data/2.5/weather?q=tiznit,ma&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 161 of Set 5 | torbay
    http://api.openweathermap.org/data/2.5/weather?q=torbay,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 162 of Set 5 | road town
    http://api.openweathermap.org/data/2.5/weather?q=road+town,vg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 163 of Set 5 | yakeshi
    http://api.openweathermap.org/data/2.5/weather?q=yakeshi,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 164 of Set 5 | khatanga
    http://api.openweathermap.org/data/2.5/weather?q=khatanga,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 165 of Set 5 | comodoro rivadavia
    http://api.openweathermap.org/data/2.5/weather?q=comodoro+rivadavia,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 166 of Set 5 | faya
    http://api.openweathermap.org/data/2.5/weather?q=faya,td&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 167 of Set 5 | boguchany
    http://api.openweathermap.org/data/2.5/weather?q=boguchany,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 168 of Set 5 | banda aceh
    http://api.openweathermap.org/data/2.5/weather?q=banda+aceh,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 169 of Set 5 | cidreira
    http://api.openweathermap.org/data/2.5/weather?q=cidreira,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 170 of Set 5 | talisay
    http://api.openweathermap.org/data/2.5/weather?q=talisay,ph&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 171 of Set 5 | mahibadhoo
    http://api.openweathermap.org/data/2.5/weather?q=mahibadhoo,mv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 172 of Set 5 | tocopilla
    http://api.openweathermap.org/data/2.5/weather?q=tocopilla,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 173 of Set 5 | barsovo
    http://api.openweathermap.org/data/2.5/weather?q=barsovo,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 174 of Set 5 | nome
    http://api.openweathermap.org/data/2.5/weather?q=nome,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 175 of Set 5 | sinnamary
    http://api.openweathermap.org/data/2.5/weather?q=sinnamary,gf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 176 of Set 5 | zhigansk
    http://api.openweathermap.org/data/2.5/weather?q=zhigansk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 177 of Set 5 | orguz
    http://api.openweathermap.org/data/2.5/weather?q=orguz,ba&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 178 of Set 5 | uyuni
    http://api.openweathermap.org/data/2.5/weather?q=uyuni,bo&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 179 of Set 5 | qaanaaq
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 180 of Set 5 | cape town
    http://api.openweathermap.org/data/2.5/weather?q=cape+town,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 181 of Set 5 | seymchan
    http://api.openweathermap.org/data/2.5/weather?q=seymchan,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 182 of Set 5 | meyungs
    http://api.openweathermap.org/data/2.5/weather?q=meyungs,pw&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 183 of Set 5 | deniliquin
    http://api.openweathermap.org/data/2.5/weather?q=deniliquin,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 184 of Set 5 | vila velha
    http://api.openweathermap.org/data/2.5/weather?q=vila+velha,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 185 of Set 5 | new norfolk
    http://api.openweathermap.org/data/2.5/weather?q=new+norfolk,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 186 of Set 5 | jinka
    http://api.openweathermap.org/data/2.5/weather?q=jinka,et&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 187 of Set 5 | pitimbu
    http://api.openweathermap.org/data/2.5/weather?q=pitimbu,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 188 of Set 5 | amderma
    http://api.openweathermap.org/data/2.5/weather?q=amderma,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 189 of Set 5 | koutsouras
    http://api.openweathermap.org/data/2.5/weather?q=koutsouras,gr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 190 of Set 5 | calama
    http://api.openweathermap.org/data/2.5/weather?q=calama,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 191 of Set 5 | baruun-urt
    http://api.openweathermap.org/data/2.5/weather?q=baruun-urt,mn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 192 of Set 5 | saint george
    http://api.openweathermap.org/data/2.5/weather?q=saint+george,bm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 193 of Set 5 | shubarkuduk
    http://api.openweathermap.org/data/2.5/weather?q=shubarkuduk,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 194 of Set 5 | butaritari
    http://api.openweathermap.org/data/2.5/weather?q=butaritari,ki&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 195 of Set 5 | novyy urengoy
    http://api.openweathermap.org/data/2.5/weather?q=novyy+urengoy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 196 of Set 5 | kavieng
    http://api.openweathermap.org/data/2.5/weather?q=kavieng,pg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 197 of Set 5 | baykit
    http://api.openweathermap.org/data/2.5/weather?q=baykit,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 198 of Set 5 | namatanai
    http://api.openweathermap.org/data/2.5/weather?q=namatanai,pg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 199 of Set 5 | makasar
    http://api.openweathermap.org/data/2.5/weather?q=makasar,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 200 of Set 6 | ixtapa
    http://api.openweathermap.org/data/2.5/weather?q=ixtapa,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 201 of Set 6 | kruisfontein
    http://api.openweathermap.org/data/2.5/weather?q=kruisfontein,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 202 of Set 6 | geraldton
    http://api.openweathermap.org/data/2.5/weather?q=geraldton,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 203 of Set 6 | oranjestad
    http://api.openweathermap.org/data/2.5/weather?q=oranjestad,an&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 204 of Set 6 | upernavik
    http://api.openweathermap.org/data/2.5/weather?q=upernavik,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 205 of Set 6 | hofn
    http://api.openweathermap.org/data/2.5/weather?q=hofn,is&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 206 of Set 6 | coquimbo
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 207 of Set 6 | zruc nad sazavou
    http://api.openweathermap.org/data/2.5/weather?q=zruc+nad+sazavou,cz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 208 of Set 6 | fort smith
    http://api.openweathermap.org/data/2.5/weather?q=fort+smith,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 209 of Set 6 | sao filipe
    http://api.openweathermap.org/data/2.5/weather?q=sao+filipe,cv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 210 of Set 6 | thompson
    http://api.openweathermap.org/data/2.5/weather?q=thompson,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 211 of Set 6 | bunbury
    http://api.openweathermap.org/data/2.5/weather?q=bunbury,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 212 of Set 6 | gamba
    http://api.openweathermap.org/data/2.5/weather?q=gamba,ga&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 213 of Set 6 | vao
    http://api.openweathermap.org/data/2.5/weather?q=vao,nc&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 214 of Set 6 | kalmunai
    http://api.openweathermap.org/data/2.5/weather?q=kalmunai,lk&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 215 of Set 6 | nanlong
    http://api.openweathermap.org/data/2.5/weather?q=nanlong,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 216 of Set 6 | yulara
    http://api.openweathermap.org/data/2.5/weather?q=yulara,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 217 of Set 6 | coruripe
    http://api.openweathermap.org/data/2.5/weather?q=coruripe,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 218 of Set 6 | tabiauea
    http://api.openweathermap.org/data/2.5/weather?q=tabiauea,ki&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 219 of Set 6 | pacific grove
    http://api.openweathermap.org/data/2.5/weather?q=pacific+grove,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 220 of Set 6 | tamandare
    http://api.openweathermap.org/data/2.5/weather?q=tamandare,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 221 of Set 6 | binghamton
    http://api.openweathermap.org/data/2.5/weather?q=binghamton,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 222 of Set 6 | chumikan
    http://api.openweathermap.org/data/2.5/weather?q=chumikan,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 223 of Set 6 | kachikau
    http://api.openweathermap.org/data/2.5/weather?q=kachikau,bw&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 224 of Set 6 | villa carlos paz
    http://api.openweathermap.org/data/2.5/weather?q=villa+carlos+paz,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 225 of Set 6 | salalah
    http://api.openweathermap.org/data/2.5/weather?q=salalah,om&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 226 of Set 6 | novopokrovka
    http://api.openweathermap.org/data/2.5/weather?q=novopokrovka,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 227 of Set 6 | am timan
    http://api.openweathermap.org/data/2.5/weather?q=am+timan,td&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 228 of Set 6 | poronaysk
    http://api.openweathermap.org/data/2.5/weather?q=poronaysk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 229 of Set 6 | havelock
    http://api.openweathermap.org/data/2.5/weather?q=havelock,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 230 of Set 6 | isangel
    http://api.openweathermap.org/data/2.5/weather?q=isangel,vu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 231 of Set 6 | atambua
    http://api.openweathermap.org/data/2.5/weather?q=atambua,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 232 of Set 6 | cherskiy
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 233 of Set 6 | mayo
    http://api.openweathermap.org/data/2.5/weather?q=mayo,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 234 of Set 6 | rio tercero
    http://api.openweathermap.org/data/2.5/weather?q=rio+tercero,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 235 of Set 6 | tortoli
    http://api.openweathermap.org/data/2.5/weather?q=tortoli,it&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 236 of Set 6 | portland
    http://api.openweathermap.org/data/2.5/weather?q=portland,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 237 of Set 6 | moroni
    http://api.openweathermap.org/data/2.5/weather?q=moroni,km&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 238 of Set 6 | tessalit
    http://api.openweathermap.org/data/2.5/weather?q=tessalit,ml&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 239 of Set 6 | vostok
    http://api.openweathermap.org/data/2.5/weather?q=vostok,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 240 of Set 7 | asfi
    http://api.openweathermap.org/data/2.5/weather?q=asfi,ma&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 241 of Set 7 | beeville
    http://api.openweathermap.org/data/2.5/weather?q=beeville,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 242 of Set 7 | mao
    http://api.openweathermap.org/data/2.5/weather?q=mao,td&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 243 of Set 7 | butembo
    http://api.openweathermap.org/data/2.5/weather?q=butembo,cd&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 244 of Set 7 | nata
    http://api.openweathermap.org/data/2.5/weather?q=nata,bw&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 245 of Set 7 | sitka
    http://api.openweathermap.org/data/2.5/weather?q=sitka,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 246 of Set 7 | benguela
    http://api.openweathermap.org/data/2.5/weather?q=benguela,ao&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 247 of Set 7 | warqla
    http://api.openweathermap.org/data/2.5/weather?q=warqla,dz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 248 of Set 7 | svetlogorsk
    http://api.openweathermap.org/data/2.5/weather?q=svetlogorsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 249 of Set 7 | temir
    http://api.openweathermap.org/data/2.5/weather?q=temir,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 250 of Set 7 | pangoa
    http://api.openweathermap.org/data/2.5/weather?q=pangoa,pe&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 251 of Set 7 | chokurdakh
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 252 of Set 7 | kommunar
    http://api.openweathermap.org/data/2.5/weather?q=kommunar,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 253 of Set 7 | quatre cocos
    http://api.openweathermap.org/data/2.5/weather?q=quatre+cocos,mu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 254 of Set 7 | roswell
    http://api.openweathermap.org/data/2.5/weather?q=roswell,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 255 of Set 7 | killam
    http://api.openweathermap.org/data/2.5/weather?q=killam,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 256 of Set 7 | dzhusaly
    http://api.openweathermap.org/data/2.5/weather?q=dzhusaly,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 257 of Set 7 | gravdal
    http://api.openweathermap.org/data/2.5/weather?q=gravdal,no&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 258 of Set 7 | sao joao da barra
    http://api.openweathermap.org/data/2.5/weather?q=sao+joao+da+barra,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 259 of Set 7 | tautira
    http://api.openweathermap.org/data/2.5/weather?q=tautira,pf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 260 of Set 7 | meulaboh
    http://api.openweathermap.org/data/2.5/weather?q=meulaboh,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 261 of Set 7 | iquitos
    http://api.openweathermap.org/data/2.5/weather?q=iquitos,pe&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 262 of Set 7 | tevaitoa
    http://api.openweathermap.org/data/2.5/weather?q=tevaitoa,pf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 263 of Set 7 | pontevedra
    http://api.openweathermap.org/data/2.5/weather?q=pontevedra,es&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 264 of Set 7 | palu
    http://api.openweathermap.org/data/2.5/weather?q=palu,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 265 of Set 7 | prince rupert
    http://api.openweathermap.org/data/2.5/weather?q=prince+rupert,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 266 of Set 7 | namtsy
    http://api.openweathermap.org/data/2.5/weather?q=namtsy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 267 of Set 7 | nizhneyansk
    http://api.openweathermap.org/data/2.5/weather?q=nizhneyansk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 268 of Set 7 | souillac
    http://api.openweathermap.org/data/2.5/weather?q=souillac,mu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 269 of Set 7 | samarai
    http://api.openweathermap.org/data/2.5/weather?q=samarai,pg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 270 of Set 7 | lorengau
    http://api.openweathermap.org/data/2.5/weather?q=lorengau,pg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 271 of Set 7 | port lincoln
    http://api.openweathermap.org/data/2.5/weather?q=port+lincoln,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 272 of Set 7 | itoman
    http://api.openweathermap.org/data/2.5/weather?q=itoman,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 273 of Set 7 | sturgis
    http://api.openweathermap.org/data/2.5/weather?q=sturgis,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 274 of Set 7 | paoua
    http://api.openweathermap.org/data/2.5/weather?q=paoua,cf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 275 of Set 7 | bajil
    http://api.openweathermap.org/data/2.5/weather?q=bajil,ye&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 276 of Set 7 | haian
    http://api.openweathermap.org/data/2.5/weather?q=haian,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 277 of Set 7 | hay river
    http://api.openweathermap.org/data/2.5/weather?q=hay+river,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 278 of Set 7 | urrutia
    http://api.openweathermap.org/data/2.5/weather?q=urrutia,hn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 279 of Set 7 | arlit
    http://api.openweathermap.org/data/2.5/weather?q=arlit,ne&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 280 of Set 8 | taunggyi
    http://api.openweathermap.org/data/2.5/weather?q=taunggyi,mm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 281 of Set 8 | freeport
    http://api.openweathermap.org/data/2.5/weather?q=freeport,bs&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 282 of Set 8 | brae
    http://api.openweathermap.org/data/2.5/weather?q=brae,gb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 283 of Set 8 | biak
    http://api.openweathermap.org/data/2.5/weather?q=biak,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 284 of Set 8 | virginia beach
    http://api.openweathermap.org/data/2.5/weather?q=virginia+beach,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 285 of Set 8 | chuy
    http://api.openweathermap.org/data/2.5/weather?q=chuy,uy&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 286 of Set 8 | bonthe
    http://api.openweathermap.org/data/2.5/weather?q=bonthe,sl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 287 of Set 8 | vallenar
    http://api.openweathermap.org/data/2.5/weather?q=vallenar,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 288 of Set 8 | abai
    http://api.openweathermap.org/data/2.5/weather?q=abai,py&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 289 of Set 8 | uglekamensk
    http://api.openweathermap.org/data/2.5/weather?q=uglekamensk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 290 of Set 8 | abu samrah
    http://api.openweathermap.org/data/2.5/weather?q=abu+samrah,qa&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 291 of Set 8 | husavik
    http://api.openweathermap.org/data/2.5/weather?q=husavik,is&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 292 of Set 8 | greenfield
    http://api.openweathermap.org/data/2.5/weather?q=greenfield,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 293 of Set 8 | adre
    http://api.openweathermap.org/data/2.5/weather?q=adre,td&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 294 of Set 8 | georgetown
    http://api.openweathermap.org/data/2.5/weather?q=georgetown,sh&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 295 of Set 8 | yicheng
    http://api.openweathermap.org/data/2.5/weather?q=yicheng,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 296 of Set 8 | muros
    http://api.openweathermap.org/data/2.5/weather?q=muros,es&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 297 of Set 8 | woodward
    http://api.openweathermap.org/data/2.5/weather?q=woodward,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 298 of Set 8 | kununurra
    http://api.openweathermap.org/data/2.5/weather?q=kununurra,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 299 of Set 8 | aswan
    http://api.openweathermap.org/data/2.5/weather?q=aswan,eg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 300 of Set 8 | ketchikan
    http://api.openweathermap.org/data/2.5/weather?q=ketchikan,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 301 of Set 8 | luwuk
    http://api.openweathermap.org/data/2.5/weather?q=luwuk,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 302 of Set 8 | tete
    http://api.openweathermap.org/data/2.5/weather?q=tete,mz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 303 of Set 8 | teul
    http://api.openweathermap.org/data/2.5/weather?q=teul,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 304 of Set 8 | palmas
    http://api.openweathermap.org/data/2.5/weather?q=palmas,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 305 of Set 8 | sao jose da coroa grande
    http://api.openweathermap.org/data/2.5/weather?q=sao+jose+da+coroa+grande,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 306 of Set 8 | cockburn town
    http://api.openweathermap.org/data/2.5/weather?q=cockburn+town,bs&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 307 of Set 8 | cairns
    http://api.openweathermap.org/data/2.5/weather?q=cairns,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 308 of Set 8 | karymskoye
    http://api.openweathermap.org/data/2.5/weather?q=karymskoye,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 309 of Set 8 | ahipara
    http://api.openweathermap.org/data/2.5/weather?q=ahipara,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 310 of Set 8 | kiama
    http://api.openweathermap.org/data/2.5/weather?q=kiama,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 311 of Set 8 | nyakahanga
    http://api.openweathermap.org/data/2.5/weather?q=nyakahanga,tz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 312 of Set 8 | curico
    http://api.openweathermap.org/data/2.5/weather?q=curico,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 313 of Set 8 | port elizabeth
    http://api.openweathermap.org/data/2.5/weather?q=port+elizabeth,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 314 of Set 8 | shingu
    http://api.openweathermap.org/data/2.5/weather?q=shingu,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 315 of Set 8 | agadir
    http://api.openweathermap.org/data/2.5/weather?q=agadir,ma&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 316 of Set 8 | oktyabrskiy
    http://api.openweathermap.org/data/2.5/weather?q=oktyabrskiy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 317 of Set 8 | alta
    http://api.openweathermap.org/data/2.5/weather?q=alta,no&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 318 of Set 8 | pavilosta
    http://api.openweathermap.org/data/2.5/weather?q=pavilosta,lv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 319 of Set 8 | barra do corda
    http://api.openweathermap.org/data/2.5/weather?q=barra+do+corda,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 320 of Set 9 | palatka
    http://api.openweathermap.org/data/2.5/weather?q=palatka,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 321 of Set 9 | lavrentiya
    http://api.openweathermap.org/data/2.5/weather?q=lavrentiya,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 322 of Set 9 | clyde river
    http://api.openweathermap.org/data/2.5/weather?q=clyde+river,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 323 of Set 9 | paamiut
    http://api.openweathermap.org/data/2.5/weather?q=paamiut,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 324 of Set 9 | syracuse
    http://api.openweathermap.org/data/2.5/weather?q=syracuse,it&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 325 of Set 9 | chapleau
    http://api.openweathermap.org/data/2.5/weather?q=chapleau,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 326 of Set 9 | cayenne
    http://api.openweathermap.org/data/2.5/weather?q=cayenne,gf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 327 of Set 9 | mbandaka
    http://api.openweathermap.org/data/2.5/weather?q=mbandaka,cd&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 328 of Set 9 | zolotinka
    http://api.openweathermap.org/data/2.5/weather?q=zolotinka,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 329 of Set 9 | cam ranh
    http://api.openweathermap.org/data/2.5/weather?q=cam+ranh,vn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 330 of Set 9 | boulder
    http://api.openweathermap.org/data/2.5/weather?q=boulder,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 331 of Set 9 | haradok
    http://api.openweathermap.org/data/2.5/weather?q=haradok,by&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 332 of Set 9 | chegutu
    http://api.openweathermap.org/data/2.5/weather?q=chegutu,zw&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 333 of Set 9 | san quintin
    http://api.openweathermap.org/data/2.5/weather?q=san+quintin,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 334 of Set 9 | bichura
    http://api.openweathermap.org/data/2.5/weather?q=bichura,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 335 of Set 9 | mnogovershinnyy
    http://api.openweathermap.org/data/2.5/weather?q=mnogovershinnyy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 336 of Set 9 | karamea
    http://api.openweathermap.org/data/2.5/weather?q=karamea,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 337 of Set 9 | mount gambier
    http://api.openweathermap.org/data/2.5/weather?q=mount+gambier,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 338 of Set 9 | makat
    http://api.openweathermap.org/data/2.5/weather?q=makat,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 339 of Set 9 | sladkovo
    http://api.openweathermap.org/data/2.5/weather?q=sladkovo,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 340 of Set 9 | effingham
    http://api.openweathermap.org/data/2.5/weather?q=effingham,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 341 of Set 9 | glamoc
    http://api.openweathermap.org/data/2.5/weather?q=glamoc,ba&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 342 of Set 9 | grand river south east
    http://api.openweathermap.org/data/2.5/weather?q=grand+river+south+east,mu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 343 of Set 9 | turkan
    http://api.openweathermap.org/data/2.5/weather?q=turkan,az&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 344 of Set 9 | tiksi
    http://api.openweathermap.org/data/2.5/weather?q=tiksi,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 345 of Set 9 | salvacion
    http://api.openweathermap.org/data/2.5/weather?q=salvacion,ph&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 346 of Set 9 | lianzhou
    http://api.openweathermap.org/data/2.5/weather?q=lianzhou,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 347 of Set 9 | vaitupu
    http://api.openweathermap.org/data/2.5/weather?q=vaitupu,wf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 348 of Set 9 | uvira
    http://api.openweathermap.org/data/2.5/weather?q=uvira,cd&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 349 of Set 9 | coahuayana
    http://api.openweathermap.org/data/2.5/weather?q=coahuayana,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 350 of Set 9 | pergamino
    http://api.openweathermap.org/data/2.5/weather?q=pergamino,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 351 of Set 9 | burica
    http://api.openweathermap.org/data/2.5/weather?q=burica,pa&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 352 of Set 9 | zhezkazgan
    http://api.openweathermap.org/data/2.5/weather?q=zhezkazgan,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 353 of Set 9 | surt
    http://api.openweathermap.org/data/2.5/weather?q=surt,ly&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 354 of Set 9 | wana
    http://api.openweathermap.org/data/2.5/weather?q=wana,pk&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 355 of Set 9 | severnyy
    http://api.openweathermap.org/data/2.5/weather?q=severnyy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 356 of Set 9 | vila franca do campo
    http://api.openweathermap.org/data/2.5/weather?q=vila+franca+do+campo,pt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 357 of Set 9 | kapoeta
    http://api.openweathermap.org/data/2.5/weather?q=kapoeta,sd&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 358 of Set 9 | wanning
    http://api.openweathermap.org/data/2.5/weather?q=wanning,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 359 of Set 9 | lufilufi
    http://api.openweathermap.org/data/2.5/weather?q=lufilufi,ws&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 360 of Set 10 | pingliang
    http://api.openweathermap.org/data/2.5/weather?q=pingliang,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 361 of Set 10 | la rochelle
    http://api.openweathermap.org/data/2.5/weather?q=la+rochelle,fr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 362 of Set 10 | tuktoyaktuk
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 363 of Set 10 | vestmannaeyjar
    http://api.openweathermap.org/data/2.5/weather?q=vestmannaeyjar,is&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 364 of Set 10 | nobres
    http://api.openweathermap.org/data/2.5/weather?q=nobres,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 365 of Set 10 | severo-kurilsk
    http://api.openweathermap.org/data/2.5/weather?q=severo-kurilsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 366 of Set 10 | port macquarie
    http://api.openweathermap.org/data/2.5/weather?q=port+macquarie,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 367 of Set 10 | yalutorovsk
    http://api.openweathermap.org/data/2.5/weather?q=yalutorovsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 368 of Set 10 | marzuq
    http://api.openweathermap.org/data/2.5/weather?q=marzuq,ly&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 369 of Set 10 | lundazi
    http://api.openweathermap.org/data/2.5/weather?q=lundazi,zm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 370 of Set 10 | saint-philippe
    http://api.openweathermap.org/data/2.5/weather?q=saint-philippe,re&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 371 of Set 10 | alice springs
    http://api.openweathermap.org/data/2.5/weather?q=alice+springs,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 372 of Set 10 | ancud
    http://api.openweathermap.org/data/2.5/weather?q=ancud,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 373 of Set 10 | dharan
    http://api.openweathermap.org/data/2.5/weather?q=dharan,np&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 374 of Set 10 | the valley
    http://api.openweathermap.org/data/2.5/weather?q=the+valley,ai&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 375 of Set 10 | bandarbeyla
    http://api.openweathermap.org/data/2.5/weather?q=bandarbeyla,so&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 376 of Set 10 | urumqi
    http://api.openweathermap.org/data/2.5/weather?q=urumqi,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 377 of Set 10 | nemuro
    http://api.openweathermap.org/data/2.5/weather?q=nemuro,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 378 of Set 10 | inegol
    http://api.openweathermap.org/data/2.5/weather?q=inegol,tr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 379 of Set 10 | sorland
    http://api.openweathermap.org/data/2.5/weather?q=sorland,no&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 380 of Set 10 | poum
    http://api.openweathermap.org/data/2.5/weather?q=poum,nc&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 381 of Set 10 | constitucion
    http://api.openweathermap.org/data/2.5/weather?q=constitucion,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 382 of Set 10 | hovd
    http://api.openweathermap.org/data/2.5/weather?q=hovd,mn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 383 of Set 10 | yeletskiy
    http://api.openweathermap.org/data/2.5/weather?q=yeletskiy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 384 of Set 10 | myitkyina
    http://api.openweathermap.org/data/2.5/weather?q=myitkyina,mm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 385 of Set 10 | mayumba
    http://api.openweathermap.org/data/2.5/weather?q=mayumba,ga&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 386 of Set 10 | skibbereen
    http://api.openweathermap.org/data/2.5/weather?q=skibbereen,ie&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 387 of Set 10 | kangavar
    http://api.openweathermap.org/data/2.5/weather?q=kangavar,ir&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 388 of Set 10 | umea
    http://api.openweathermap.org/data/2.5/weather?q=umea,se&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 389 of Set 10 | koratla
    http://api.openweathermap.org/data/2.5/weather?q=koratla,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 390 of Set 10 | altay
    http://api.openweathermap.org/data/2.5/weather?q=altay,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 391 of Set 10 | ariquemes
    http://api.openweathermap.org/data/2.5/weather?q=ariquemes,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 392 of Set 10 | kijang
    http://api.openweathermap.org/data/2.5/weather?q=kijang,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 393 of Set 10 | yago
    http://api.openweathermap.org/data/2.5/weather?q=yago,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 394 of Set 10 | leningradskiy
    http://api.openweathermap.org/data/2.5/weather?q=leningradskiy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 395 of Set 10 | jacmel
    http://api.openweathermap.org/data/2.5/weather?q=jacmel,ht&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 396 of Set 10 | rocha
    http://api.openweathermap.org/data/2.5/weather?q=rocha,uy&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 397 of Set 10 | muisne
    http://api.openweathermap.org/data/2.5/weather?q=muisne,ec&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 398 of Set 10 | luderitz
    http://api.openweathermap.org/data/2.5/weather?q=luderitz,na&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 399 of Set 10 | primorsko-akhtarsk
    http://api.openweathermap.org/data/2.5/weather?q=primorsko-akhtarsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 400 of Set 11 | nizhniy chir
    http://api.openweathermap.org/data/2.5/weather?q=nizhniy+chir,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 401 of Set 11 | flinders
    http://api.openweathermap.org/data/2.5/weather?q=flinders,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 402 of Set 11 | yambio
    http://api.openweathermap.org/data/2.5/weather?q=yambio,sd&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 403 of Set 11 | nguiu
    http://api.openweathermap.org/data/2.5/weather?q=nguiu,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 404 of Set 11 | belmonte
    http://api.openweathermap.org/data/2.5/weather?q=belmonte,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 405 of Set 11 | rosetown
    http://api.openweathermap.org/data/2.5/weather?q=rosetown,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 406 of Set 11 | gangotri
    http://api.openweathermap.org/data/2.5/weather?q=gangotri,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 407 of Set 11 | barguzin
    http://api.openweathermap.org/data/2.5/weather?q=barguzin,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 408 of Set 11 | san andres
    http://api.openweathermap.org/data/2.5/weather?q=san+andres,co&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 409 of Set 11 | yekaterinogradskaya
    http://api.openweathermap.org/data/2.5/weather?q=yekaterinogradskaya,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 410 of Set 11 | sarlat-la-caneda
    http://api.openweathermap.org/data/2.5/weather?q=sarlat-la-caneda,fr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 411 of Set 11 | honiara
    http://api.openweathermap.org/data/2.5/weather?q=honiara,sb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 412 of Set 11 | saint anthony
    http://api.openweathermap.org/data/2.5/weather?q=saint+anthony,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 413 of Set 11 | kolondieba
    http://api.openweathermap.org/data/2.5/weather?q=kolondieba,ml&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 414 of Set 11 | kristiinankaupunki
    http://api.openweathermap.org/data/2.5/weather?q=kristiinankaupunki,fi&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 415 of Set 11 | hendijan
    http://api.openweathermap.org/data/2.5/weather?q=hendijan,ir&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 416 of Set 11 | mindelo
    http://api.openweathermap.org/data/2.5/weather?q=mindelo,cv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 417 of Set 11 | yirol
    http://api.openweathermap.org/data/2.5/weather?q=yirol,sd&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 418 of Set 11 | luxor
    http://api.openweathermap.org/data/2.5/weather?q=luxor,eg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 419 of Set 11 | ballina
    http://api.openweathermap.org/data/2.5/weather?q=ballina,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 420 of Set 11 | beckerich
    http://api.openweathermap.org/data/2.5/weather?q=beckerich,lu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 421 of Set 11 | limon
    http://api.openweathermap.org/data/2.5/weather?q=limon,hn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 422 of Set 11 | andevoranto
    http://api.openweathermap.org/data/2.5/weather?q=andevoranto,mg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 423 of Set 11 | platanos
    http://api.openweathermap.org/data/2.5/weather?q=platanos,gr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 424 of Set 11 | ucluelet
    http://api.openweathermap.org/data/2.5/weather?q=ucluelet,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 425 of Set 11 | shahrud
    http://api.openweathermap.org/data/2.5/weather?q=shahrud,ir&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 426 of Set 11 | opuwo
    http://api.openweathermap.org/data/2.5/weather?q=opuwo,na&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 427 of Set 11 | naze
    http://api.openweathermap.org/data/2.5/weather?q=naze,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 428 of Set 11 | elvas
    http://api.openweathermap.org/data/2.5/weather?q=elvas,pt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 429 of Set 11 | itarema
    http://api.openweathermap.org/data/2.5/weather?q=itarema,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 430 of Set 11 | narsaq
    http://api.openweathermap.org/data/2.5/weather?q=narsaq,gl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 431 of Set 11 | santarem
    http://api.openweathermap.org/data/2.5/weather?q=santarem,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 432 of Set 11 | rudnogorsk
    http://api.openweathermap.org/data/2.5/weather?q=rudnogorsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 433 of Set 11 | harper
    http://api.openweathermap.org/data/2.5/weather?q=harper,lr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 434 of Set 11 | aksarka
    http://api.openweathermap.org/data/2.5/weather?q=aksarka,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 435 of Set 11 | ribas do rio pardo
    http://api.openweathermap.org/data/2.5/weather?q=ribas+do+rio+pardo,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 436 of Set 11 | nabire
    http://api.openweathermap.org/data/2.5/weather?q=nabire,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 437 of Set 11 | whitehorse
    http://api.openweathermap.org/data/2.5/weather?q=whitehorse,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 438 of Set 11 | motygino
    http://api.openweathermap.org/data/2.5/weather?q=motygino,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 439 of Set 11 | sao joao
    http://api.openweathermap.org/data/2.5/weather?q=sao+joao,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 440 of Set 12 | kanniyakumari
    http://api.openweathermap.org/data/2.5/weather?q=kanniyakumari,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 441 of Set 12 | benghazi
    http://api.openweathermap.org/data/2.5/weather?q=benghazi,ly&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 442 of Set 12 | fort nelson
    http://api.openweathermap.org/data/2.5/weather?q=fort+nelson,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 443 of Set 12 | ahuimanu
    http://api.openweathermap.org/data/2.5/weather?q=ahuimanu,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 444 of Set 12 | aga
    http://api.openweathermap.org/data/2.5/weather?q=aga,ph&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 445 of Set 12 | kathmandu
    http://api.openweathermap.org/data/2.5/weather?q=kathmandu,np&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 446 of Set 12 | boa vista
    http://api.openweathermap.org/data/2.5/weather?q=boa+vista,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 447 of Set 12 | vigia del fuerte
    http://api.openweathermap.org/data/2.5/weather?q=vigia+del+fuerte,co&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 448 of Set 12 | goderich
    http://api.openweathermap.org/data/2.5/weather?q=goderich,sl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 449 of Set 12 | plettenberg bay
    http://api.openweathermap.org/data/2.5/weather?q=plettenberg+bay,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 450 of Set 12 | norman wells
    http://api.openweathermap.org/data/2.5/weather?q=norman+wells,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 451 of Set 12 | scarborough
    http://api.openweathermap.org/data/2.5/weather?q=scarborough,tt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 452 of Set 12 | barddhaman
    http://api.openweathermap.org/data/2.5/weather?q=barddhaman,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 453 of Set 12 | yar-sale
    http://api.openweathermap.org/data/2.5/weather?q=yar-sale,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 454 of Set 12 | gornopravdinsk
    http://api.openweathermap.org/data/2.5/weather?q=gornopravdinsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 455 of Set 12 | helena
    http://api.openweathermap.org/data/2.5/weather?q=helena,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 456 of Set 12 | katherine
    http://api.openweathermap.org/data/2.5/weather?q=katherine,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 457 of Set 12 | bose
    http://api.openweathermap.org/data/2.5/weather?q=bose,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 458 of Set 12 | dromolaxia
    http://api.openweathermap.org/data/2.5/weather?q=dromolaxia,cy&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 459 of Set 12 | mairana
    http://api.openweathermap.org/data/2.5/weather?q=mairana,bo&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 460 of Set 12 | monrovia
    http://api.openweathermap.org/data/2.5/weather?q=monrovia,lr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 461 of Set 12 | brigantine
    http://api.openweathermap.org/data/2.5/weather?q=brigantine,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 462 of Set 12 | yartsevo
    http://api.openweathermap.org/data/2.5/weather?q=yartsevo,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 463 of Set 12 | lima
    http://api.openweathermap.org/data/2.5/weather?q=lima,pe&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 464 of Set 12 | nandurbar
    http://api.openweathermap.org/data/2.5/weather?q=nandurbar,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 465 of Set 12 | yumen
    http://api.openweathermap.org/data/2.5/weather?q=yumen,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 466 of Set 12 | carballo
    http://api.openweathermap.org/data/2.5/weather?q=carballo,es&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 467 of Set 12 | hobyo
    http://api.openweathermap.org/data/2.5/weather?q=hobyo,so&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 468 of Set 12 | kirkwall
    http://api.openweathermap.org/data/2.5/weather?q=kirkwall,gb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 469 of Set 12 | george town
    http://api.openweathermap.org/data/2.5/weather?q=george+town,ky&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 470 of Set 12 | nouadhibou
    http://api.openweathermap.org/data/2.5/weather?q=nouadhibou,mr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 471 of Set 12 | liaozhong
    http://api.openweathermap.org/data/2.5/weather?q=liaozhong,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 472 of Set 12 | auki
    http://api.openweathermap.org/data/2.5/weather?q=auki,sb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 473 of Set 12 | safford
    http://api.openweathermap.org/data/2.5/weather?q=safford,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 474 of Set 12 | ponta delgada
    http://api.openweathermap.org/data/2.5/weather?q=ponta+delgada,pt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 475 of Set 12 | anshun
    http://api.openweathermap.org/data/2.5/weather?q=anshun,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 476 of Set 12 | marienburg
    http://api.openweathermap.org/data/2.5/weather?q=marienburg,sr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 477 of Set 12 | san ramon
    http://api.openweathermap.org/data/2.5/weather?q=san+ramon,bo&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 478 of Set 12 | kovdor
    http://api.openweathermap.org/data/2.5/weather?q=kovdor,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 479 of Set 12 | mocuba
    http://api.openweathermap.org/data/2.5/weather?q=mocuba,mz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 480 of Set 13 | sept-iles
    http://api.openweathermap.org/data/2.5/weather?q=sept-iles,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 481 of Set 13 | tomari
    http://api.openweathermap.org/data/2.5/weather?q=tomari,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 482 of Set 13 | halalo
    http://api.openweathermap.org/data/2.5/weather?q=halalo,wf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 483 of Set 13 | nipawin
    http://api.openweathermap.org/data/2.5/weather?q=nipawin,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 484 of Set 13 | sonari
    http://api.openweathermap.org/data/2.5/weather?q=sonari,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 485 of Set 13 | riverton
    http://api.openweathermap.org/data/2.5/weather?q=riverton,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 486 of Set 13 | puerto madryn
    http://api.openweathermap.org/data/2.5/weather?q=puerto+madryn,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 487 of Set 13 | rio verde de mato grosso
    http://api.openweathermap.org/data/2.5/weather?q=rio+verde+de+mato+grosso,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 488 of Set 13 | ambilobe
    http://api.openweathermap.org/data/2.5/weather?q=ambilobe,mg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 489 of Set 13 | searcy
    http://api.openweathermap.org/data/2.5/weather?q=searcy,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 490 of Set 13 | camacha
    http://api.openweathermap.org/data/2.5/weather?q=camacha,pt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 491 of Set 13 | shelbyville
    http://api.openweathermap.org/data/2.5/weather?q=shelbyville,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 492 of Set 13 | point fortin
    http://api.openweathermap.org/data/2.5/weather?q=point+fortin,tt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 493 of Set 13 | birjand
    http://api.openweathermap.org/data/2.5/weather?q=birjand,ir&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 494 of Set 13 | eldikan
    http://api.openweathermap.org/data/2.5/weather?q=eldikan,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 495 of Set 13 | san ignacio
    http://api.openweathermap.org/data/2.5/weather?q=san+ignacio,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 496 of Set 13 | yatou
    http://api.openweathermap.org/data/2.5/weather?q=yatou,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 497 of Set 13 | kudahuvadhoo
    http://api.openweathermap.org/data/2.5/weather?q=kudahuvadhoo,mv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 498 of Set 13 | abha
    http://api.openweathermap.org/data/2.5/weather?q=abha,sa&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 499 of Set 13 | coreau
    http://api.openweathermap.org/data/2.5/weather?q=coreau,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 500 of Set 13 | teya
    http://api.openweathermap.org/data/2.5/weather?q=teya,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 501 of Set 13 | yurginskoye
    http://api.openweathermap.org/data/2.5/weather?q=yurginskoye,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 502 of Set 13 | meiganga
    http://api.openweathermap.org/data/2.5/weather?q=meiganga,cm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 503 of Set 13 | deputatskiy
    http://api.openweathermap.org/data/2.5/weather?q=deputatskiy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 504 of Set 13 | haifa
    http://api.openweathermap.org/data/2.5/weather?q=haifa,il&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 505 of Set 13 | tumut
    http://api.openweathermap.org/data/2.5/weather?q=tumut,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 506 of Set 13 | praia da vitoria
    http://api.openweathermap.org/data/2.5/weather?q=praia+da+vitoria,pt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 507 of Set 13 | half moon bay
    http://api.openweathermap.org/data/2.5/weather?q=half+moon+bay,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 508 of Set 13 | talnakh
    http://api.openweathermap.org/data/2.5/weather?q=talnakh,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 509 of Set 13 | sucha beskidzka
    http://api.openweathermap.org/data/2.5/weather?q=sucha+beskidzka,pl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 510 of Set 13 | nantucket
    http://api.openweathermap.org/data/2.5/weather?q=nantucket,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 511 of Set 13 | saleaula
    http://api.openweathermap.org/data/2.5/weather?q=saleaula,ws&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 512 of Set 13 | ilam
    http://api.openweathermap.org/data/2.5/weather?q=ilam,ir&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 513 of Set 13 | tura
    http://api.openweathermap.org/data/2.5/weather?q=tura,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 514 of Set 13 | lincoln
    http://api.openweathermap.org/data/2.5/weather?q=lincoln,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 515 of Set 13 | high level
    http://api.openweathermap.org/data/2.5/weather?q=high+level,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 516 of Set 13 | kangalassy
    http://api.openweathermap.org/data/2.5/weather?q=kangalassy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 517 of Set 13 | adrar
    http://api.openweathermap.org/data/2.5/weather?q=adrar,dz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 518 of Set 13 | krasnodon
    http://api.openweathermap.org/data/2.5/weather?q=krasnodon,ua&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 519 of Set 13 | jalingo
    http://api.openweathermap.org/data/2.5/weather?q=jalingo,ng&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 520 of Set 14 | lagos
    http://api.openweathermap.org/data/2.5/weather?q=lagos,ng&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 521 of Set 14 | aklavik
    http://api.openweathermap.org/data/2.5/weather?q=aklavik,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 522 of Set 14 | morehead
    http://api.openweathermap.org/data/2.5/weather?q=morehead,pg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 523 of Set 14 | potgietersrus
    http://api.openweathermap.org/data/2.5/weather?q=potgietersrus,za&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 524 of Set 14 | talpa
    http://api.openweathermap.org/data/2.5/weather?q=talpa,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 525 of Set 14 | jiddah
    http://api.openweathermap.org/data/2.5/weather?q=jiddah,sa&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 526 of Set 14 | sibolga
    http://api.openweathermap.org/data/2.5/weather?q=sibolga,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 527 of Set 14 | pochutla
    http://api.openweathermap.org/data/2.5/weather?q=pochutla,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 528 of Set 14 | carutapera
    http://api.openweathermap.org/data/2.5/weather?q=carutapera,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 529 of Set 14 | labuhan
    http://api.openweathermap.org/data/2.5/weather?q=labuhan,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 530 of Set 14 | bridlington
    http://api.openweathermap.org/data/2.5/weather?q=bridlington,gb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 531 of Set 14 | grants pass
    http://api.openweathermap.org/data/2.5/weather?q=grants+pass,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 532 of Set 14 | saint-pierre
    http://api.openweathermap.org/data/2.5/weather?q=saint-pierre,pm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 533 of Set 14 | ruatoria
    http://api.openweathermap.org/data/2.5/weather?q=ruatoria,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 534 of Set 14 | oksfjord
    http://api.openweathermap.org/data/2.5/weather?q=oksfjord,no&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 535 of Set 14 | almeirim
    http://api.openweathermap.org/data/2.5/weather?q=almeirim,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 536 of Set 14 | tibati
    http://api.openweathermap.org/data/2.5/weather?q=tibati,cm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 537 of Set 14 | aksu
    http://api.openweathermap.org/data/2.5/weather?q=aksu,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 538 of Set 14 | waipawa
    http://api.openweathermap.org/data/2.5/weather?q=waipawa,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 539 of Set 14 | semey
    http://api.openweathermap.org/data/2.5/weather?q=semey,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 540 of Set 14 | phan rang
    http://api.openweathermap.org/data/2.5/weather?q=phan+rang,vn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 541 of Set 14 | turayf
    http://api.openweathermap.org/data/2.5/weather?q=turayf,sa&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 542 of Set 14 | gurskoye
    http://api.openweathermap.org/data/2.5/weather?q=gurskoye,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 543 of Set 14 | okha
    http://api.openweathermap.org/data/2.5/weather?q=okha,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 544 of Set 14 | samalaeulu
    http://api.openweathermap.org/data/2.5/weather?q=samalaeulu,ws&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 545 of Set 14 | rapla
    http://api.openweathermap.org/data/2.5/weather?q=rapla,ee&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 546 of Set 14 | alofi
    http://api.openweathermap.org/data/2.5/weather?q=alofi,nu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 547 of Set 14 | morant bay
    http://api.openweathermap.org/data/2.5/weather?q=morant+bay,jm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 548 of Set 14 | marystown
    http://api.openweathermap.org/data/2.5/weather?q=marystown,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 549 of Set 14 | inirida
    http://api.openweathermap.org/data/2.5/weather?q=inirida,co&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 550 of Set 14 | rosetta
    http://api.openweathermap.org/data/2.5/weather?q=rosetta,eg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 551 of Set 14 | nisia floresta
    http://api.openweathermap.org/data/2.5/weather?q=nisia+floresta,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 552 of Set 14 | san pedro
    http://api.openweathermap.org/data/2.5/weather?q=san+pedro,bo&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 553 of Set 14 | nanga eboko
    http://api.openweathermap.org/data/2.5/weather?q=nanga+eboko,cm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 554 of Set 14 | bontang
    http://api.openweathermap.org/data/2.5/weather?q=bontang,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 555 of Set 14 | barkot
    http://api.openweathermap.org/data/2.5/weather?q=barkot,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 556 of Set 14 | lunino
    http://api.openweathermap.org/data/2.5/weather?q=lunino,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 557 of Set 14 | worland
    http://api.openweathermap.org/data/2.5/weather?q=worland,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 558 of Set 14 | harrison
    http://api.openweathermap.org/data/2.5/weather?q=harrison,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 559 of Set 14 | pinczow
    http://api.openweathermap.org/data/2.5/weather?q=pinczow,pl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 560 of Set 15 | sidrolandia
    http://api.openweathermap.org/data/2.5/weather?q=sidrolandia,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 561 of Set 15 | matara
    http://api.openweathermap.org/data/2.5/weather?q=matara,lk&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 562 of Set 15 | nanakuli
    http://api.openweathermap.org/data/2.5/weather?q=nanakuli,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 563 of Set 15 | ibaiti
    http://api.openweathermap.org/data/2.5/weather?q=ibaiti,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 564 of Set 15 | gavle
    http://api.openweathermap.org/data/2.5/weather?q=gavle,se&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 565 of Set 15 | vanimo
    http://api.openweathermap.org/data/2.5/weather?q=vanimo,pg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 566 of Set 15 | barinitas
    http://api.openweathermap.org/data/2.5/weather?q=barinitas,ve&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 567 of Set 15 | port hardy
    http://api.openweathermap.org/data/2.5/weather?q=port+hardy,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 568 of Set 15 | neon karlovasion
    http://api.openweathermap.org/data/2.5/weather?q=neon+karlovasion,gr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 569 of Set 15 | henties bay
    http://api.openweathermap.org/data/2.5/weather?q=henties+bay,na&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 570 of Set 15 | kahului
    http://api.openweathermap.org/data/2.5/weather?q=kahului,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 571 of Set 15 | bay roberts
    http://api.openweathermap.org/data/2.5/weather?q=bay+roberts,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 572 of Set 15 | marcona
    http://api.openweathermap.org/data/2.5/weather?q=marcona,pe&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 573 of Set 15 | sabha
    http://api.openweathermap.org/data/2.5/weather?q=sabha,ly&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 574 of Set 15 | kupang
    http://api.openweathermap.org/data/2.5/weather?q=kupang,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 575 of Set 15 | sofiysk
    http://api.openweathermap.org/data/2.5/weather?q=sofiysk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 576 of Set 15 | lompoc
    http://api.openweathermap.org/data/2.5/weather?q=lompoc,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 577 of Set 15 | dutse
    http://api.openweathermap.org/data/2.5/weather?q=dutse,ng&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 578 of Set 15 | nakasongola
    http://api.openweathermap.org/data/2.5/weather?q=nakasongola,ug&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 579 of Set 15 | nsanje
    http://api.openweathermap.org/data/2.5/weather?q=nsanje,mw&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 580 of Set 15 | vardo
    http://api.openweathermap.org/data/2.5/weather?q=vardo,no&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 581 of Set 15 | sfantu gheorghe
    http://api.openweathermap.org/data/2.5/weather?q=sfantu+gheorghe,ro&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 582 of Set 15 | bilibino
    http://api.openweathermap.org/data/2.5/weather?q=bilibino,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 583 of Set 15 | dhidhdhoo
    http://api.openweathermap.org/data/2.5/weather?q=dhidhdhoo,mv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 584 of Set 15 | yaan
    http://api.openweathermap.org/data/2.5/weather?q=yaan,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 585 of Set 15 | khash
    http://api.openweathermap.org/data/2.5/weather?q=khash,ir&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 586 of Set 15 | mount isa
    http://api.openweathermap.org/data/2.5/weather?q=mount+isa,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 587 of Set 15 | chazuta
    http://api.openweathermap.org/data/2.5/weather?q=chazuta,pe&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 588 of Set 15 | tuggurt
    http://api.openweathermap.org/data/2.5/weather?q=tuggurt,dz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 589 of Set 15 | kutum
    http://api.openweathermap.org/data/2.5/weather?q=kutum,sd&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 590 of Set 15 | trairi
    http://api.openweathermap.org/data/2.5/weather?q=trairi,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 591 of Set 15 | saint-augustin
    http://api.openweathermap.org/data/2.5/weather?q=saint-augustin,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 592 of Set 15 | valleyview
    http://api.openweathermap.org/data/2.5/weather?q=valleyview,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 593 of Set 15 | moberly
    http://api.openweathermap.org/data/2.5/weather?q=moberly,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 594 of Set 15 | mandiana
    http://api.openweathermap.org/data/2.5/weather?q=mandiana,gn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 595 of Set 15 | norden
    http://api.openweathermap.org/data/2.5/weather?q=norden,de&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 596 of Set 15 | san cristobal
    http://api.openweathermap.org/data/2.5/weather?q=san+cristobal,ec&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 597 of Set 15 | brasilia
    http://api.openweathermap.org/data/2.5/weather?q=brasilia,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 598 of Set 15 | huntsville
    http://api.openweathermap.org/data/2.5/weather?q=huntsville,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 599 of Set 15 | temozon
    http://api.openweathermap.org/data/2.5/weather?q=temozon,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 600 of Set 16 | tarudant
    http://api.openweathermap.org/data/2.5/weather?q=tarudant,ma&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 601 of Set 16 | alexandria
    http://api.openweathermap.org/data/2.5/weather?q=alexandria,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 602 of Set 16 | la paz
    http://api.openweathermap.org/data/2.5/weather?q=la+paz,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 603 of Set 16 | poykovskiy
    http://api.openweathermap.org/data/2.5/weather?q=poykovskiy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 604 of Set 16 | barinas
    http://api.openweathermap.org/data/2.5/weather?q=barinas,ve&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 605 of Set 16 | popondetta
    http://api.openweathermap.org/data/2.5/weather?q=popondetta,pg&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 606 of Set 16 | rundu
    http://api.openweathermap.org/data/2.5/weather?q=rundu,na&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 607 of Set 16 | poso
    http://api.openweathermap.org/data/2.5/weather?q=poso,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 608 of Set 16 | atar
    http://api.openweathermap.org/data/2.5/weather?q=atar,mr&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 609 of Set 16 | mehamn
    http://api.openweathermap.org/data/2.5/weather?q=mehamn,no&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 610 of Set 16 | manggar
    http://api.openweathermap.org/data/2.5/weather?q=manggar,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 611 of Set 16 | dukat
    http://api.openweathermap.org/data/2.5/weather?q=dukat,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 612 of Set 16 | payo
    http://api.openweathermap.org/data/2.5/weather?q=payo,ph&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 613 of Set 16 | ostrovnoy
    http://api.openweathermap.org/data/2.5/weather?q=ostrovnoy,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 614 of Set 16 | emerald
    http://api.openweathermap.org/data/2.5/weather?q=emerald,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 615 of Set 16 | moranbah
    http://api.openweathermap.org/data/2.5/weather?q=moranbah,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 616 of Set 16 | pasighat
    http://api.openweathermap.org/data/2.5/weather?q=pasighat,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 617 of Set 16 | vaitape
    http://api.openweathermap.org/data/2.5/weather?q=vaitape,pf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 618 of Set 16 | dzitankov
    http://api.openweathermap.org/data/2.5/weather?q=dzitankov,am&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 619 of Set 16 | madera
    http://api.openweathermap.org/data/2.5/weather?q=madera,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 620 of Set 16 | guaratinga
    http://api.openweathermap.org/data/2.5/weather?q=guaratinga,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 621 of Set 16 | nuevo progreso
    http://api.openweathermap.org/data/2.5/weather?q=nuevo+progreso,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 622 of Set 16 | parry sound
    http://api.openweathermap.org/data/2.5/weather?q=parry+sound,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 623 of Set 16 | flin flon
    http://api.openweathermap.org/data/2.5/weather?q=flin+flon,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 624 of Set 16 | foz
    http://api.openweathermap.org/data/2.5/weather?q=foz,es&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 625 of Set 16 | sola
    http://api.openweathermap.org/data/2.5/weather?q=sola,vu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 626 of Set 16 | bahia blanca
    http://api.openweathermap.org/data/2.5/weather?q=bahia+blanca,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 627 of Set 16 | bourail
    http://api.openweathermap.org/data/2.5/weather?q=bourail,nc&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 628 of Set 16 | tirat karmel
    http://api.openweathermap.org/data/2.5/weather?q=tirat+karmel,il&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 629 of Set 16 | rungata
    http://api.openweathermap.org/data/2.5/weather?q=rungata,ki&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 630 of Set 16 | broome
    http://api.openweathermap.org/data/2.5/weather?q=broome,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 631 of Set 16 | erdenet
    http://api.openweathermap.org/data/2.5/weather?q=erdenet,mn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 632 of Set 16 | prince george
    http://api.openweathermap.org/data/2.5/weather?q=prince+george,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 633 of Set 16 | boende
    http://api.openweathermap.org/data/2.5/weather?q=boende,cd&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 634 of Set 16 | abalak
    http://api.openweathermap.org/data/2.5/weather?q=abalak,ne&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 635 of Set 16 | luis correia
    http://api.openweathermap.org/data/2.5/weather?q=luis+correia,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 636 of Set 16 | matagami
    http://api.openweathermap.org/data/2.5/weather?q=matagami,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 637 of Set 16 | camana
    http://api.openweathermap.org/data/2.5/weather?q=camana,pe&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 638 of Set 16 | diego de almagro
    http://api.openweathermap.org/data/2.5/weather?q=diego+de+almagro,cl&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 639 of Set 16 | cap malheureux
    http://api.openweathermap.org/data/2.5/weather?q=cap+malheureux,mu&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 640 of Set 17 | wulanhaote
    http://api.openweathermap.org/data/2.5/weather?q=wulanhaote,cn&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 641 of Set 17 | dawei
    http://api.openweathermap.org/data/2.5/weather?q=dawei,mm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 642 of Set 17 | crab hill
    http://api.openweathermap.org/data/2.5/weather?q=crab+hill,bb&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 643 of Set 17 | keflavik
    http://api.openweathermap.org/data/2.5/weather?q=keflavik,is&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 644 of Set 17 | north platte
    http://api.openweathermap.org/data/2.5/weather?q=north+platte,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 645 of Set 17 | kaseda
    http://api.openweathermap.org/data/2.5/weather?q=kaseda,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 646 of Set 17 | zheleznogorsk
    http://api.openweathermap.org/data/2.5/weather?q=zheleznogorsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 647 of Set 17 | mabuhay
    http://api.openweathermap.org/data/2.5/weather?q=mabuhay,ph&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 648 of Set 17 | itacarambi
    http://api.openweathermap.org/data/2.5/weather?q=itacarambi,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 649 of Set 17 | pendleton
    http://api.openweathermap.org/data/2.5/weather?q=pendleton,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 650 of Set 17 | arrifes
    http://api.openweathermap.org/data/2.5/weather?q=arrifes,pt&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 651 of Set 17 | ouadda
    http://api.openweathermap.org/data/2.5/weather?q=ouadda,cf&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 652 of Set 17 | fallon
    http://api.openweathermap.org/data/2.5/weather?q=fallon,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 653 of Set 17 | amahai
    http://api.openweathermap.org/data/2.5/weather?q=amahai,id&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 654 of Set 17 | mumbwa
    http://api.openweathermap.org/data/2.5/weather?q=mumbwa,zm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 655 of Set 17 | nguru
    http://api.openweathermap.org/data/2.5/weather?q=nguru,ng&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 656 of Set 17 | rosario
    http://api.openweathermap.org/data/2.5/weather?q=rosario,ar&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 657 of Set 17 | charters towers
    http://api.openweathermap.org/data/2.5/weather?q=charters+towers,au&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 658 of Set 17 | mporokoso
    http://api.openweathermap.org/data/2.5/weather?q=mporokoso,zm&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 659 of Set 17 | marsabit
    http://api.openweathermap.org/data/2.5/weather?q=marsabit,ke&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 660 of Set 17 | evensk
    http://api.openweathermap.org/data/2.5/weather?q=evensk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 661 of Set 17 | moyale
    http://api.openweathermap.org/data/2.5/weather?q=moyale,ke&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 662 of Set 17 | preston
    http://api.openweathermap.org/data/2.5/weather?q=preston,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 663 of Set 17 | sisophon
    http://api.openweathermap.org/data/2.5/weather?q=sisophon,kh&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 664 of Set 17 | tlacotalpan
    http://api.openweathermap.org/data/2.5/weather?q=tlacotalpan,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 665 of Set 17 | coralville
    http://api.openweathermap.org/data/2.5/weather?q=coralville,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 666 of Set 17 | shimizu
    http://api.openweathermap.org/data/2.5/weather?q=shimizu,jp&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 667 of Set 17 | jawar
    http://api.openweathermap.org/data/2.5/weather?q=jawar,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 668 of Set 17 | hamois
    http://api.openweathermap.org/data/2.5/weather?q=hamois,be&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 669 of Set 17 | arkhangelsk
    http://api.openweathermap.org/data/2.5/weather?q=arkhangelsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 670 of Set 17 | sal rei
    http://api.openweathermap.org/data/2.5/weather?q=sal+rei,cv&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 671 of Set 17 | kelso
    http://api.openweathermap.org/data/2.5/weather?q=kelso,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 672 of Set 17 | matamoros
    http://api.openweathermap.org/data/2.5/weather?q=matamoros,mx&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 673 of Set 17 | gaya
    http://api.openweathermap.org/data/2.5/weather?q=gaya,ne&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 674 of Set 17 | snyder
    http://api.openweathermap.org/data/2.5/weather?q=snyder,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 675 of Set 17 | verkh-usugli
    http://api.openweathermap.org/data/2.5/weather?q=verkh-usugli,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 676 of Set 17 | palana
    http://api.openweathermap.org/data/2.5/weather?q=palana,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 677 of Set 17 | manoel urbano
    http://api.openweathermap.org/data/2.5/weather?q=manoel+urbano,br&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 678 of Set 17 | krabi
    http://api.openweathermap.org/data/2.5/weather?q=krabi,th&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 679 of Set 17 | winslow
    http://api.openweathermap.org/data/2.5/weather?q=winslow,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    ~~~ Break Time ~~~
    
    Processing Record 680 of Set 18 | izhevsk
    http://api.openweathermap.org/data/2.5/weather?q=izhevsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 681 of Set 18 | mahon
    http://api.openweathermap.org/data/2.5/weather?q=mahon,es&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 682 of Set 18 | bratsk
    http://api.openweathermap.org/data/2.5/weather?q=bratsk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 683 of Set 18 | uddevalla
    http://api.openweathermap.org/data/2.5/weather?q=uddevalla,se&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 684 of Set 18 | saryozek
    http://api.openweathermap.org/data/2.5/weather?q=saryozek,kz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 685 of Set 18 | winnemucca
    http://api.openweathermap.org/data/2.5/weather?q=winnemucca,us&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 686 of Set 18 | mundra
    http://api.openweathermap.org/data/2.5/weather?q=mundra,in&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 687 of Set 18 | sioux lookout
    http://api.openweathermap.org/data/2.5/weather?q=sioux+lookout,ca&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 688 of Set 18 | bintulu
    http://api.openweathermap.org/data/2.5/weather?q=bintulu,my&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 689 of Set 18 | opunake
    http://api.openweathermap.org/data/2.5/weather?q=opunake,nz&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 690 of Set 18 | skaelskor
    http://api.openweathermap.org/data/2.5/weather?q=skaelskor,dk&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 691 of Set 18 | bella union
    http://api.openweathermap.org/data/2.5/weather?q=bella+union,uy&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 692 of Set 18 | kirgiz-miyaki
    http://api.openweathermap.org/data/2.5/weather?q=kirgiz-miyaki,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 693 of Set 18 | vilyuysk
    http://api.openweathermap.org/data/2.5/weather?q=vilyuysk,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    Processing Record 694 of Set 18 | chagda
    http://api.openweathermap.org/data/2.5/weather?q=chagda,ru&appid=34ae2627193903bb86906bf116b20b46&units=imperial
    


```python
# Remove any rows that did not have data and run a final count to ensure a large enough data table
final_df = list1_df.loc[list1_df["Temp"] != "FAIL", :]
print(final_df.count())
final_df.head()
```

    Lat        607
    Lng        607
    City       607
    Country    607
    Temp       607
    Hum        607
    Cloud      607
    Wind       607
    dtype: int64
    




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Lat</th>
      <th>Lng</th>
      <th>City</th>
      <th>Country</th>
      <th>Temp</th>
      <th>Hum</th>
      <th>Cloud</th>
      <th>Wind</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-8.425875</td>
      <td>-113.445745</td>
      <td>puerto ayora</td>
      <td>ec</td>
      <td>75.2</td>
      <td>69</td>
      <td>75</td>
      <td>10.29</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-70.629471</td>
      <td>-25.455300</td>
      <td>ushuaia</td>
      <td>ar</td>
      <td>50</td>
      <td>57</td>
      <td>90</td>
      <td>12.75</td>
    </tr>
    <tr>
      <th>3</th>
      <td>50.397708</td>
      <td>114.131234</td>
      <td>duldurga</td>
      <td>ru</td>
      <td>5.44</td>
      <td>53</td>
      <td>12</td>
      <td>10.09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>41.792746</td>
      <td>177.089244</td>
      <td>nikolskoye</td>
      <td>ru</td>
      <td>28.4</td>
      <td>92</td>
      <td>90</td>
      <td>17.9</td>
    </tr>
    <tr>
      <th>6</th>
      <td>77.889262</td>
      <td>-3.535988</td>
      <td>klaksvik</td>
      <td>fo</td>
      <td>42.8</td>
      <td>81</td>
      <td>32</td>
      <td>23.04</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Make new df's to plot
temp_df = final_df[["Lat", "Temp"]]
hum_df = final_df[["Lat", "Hum"]]
cloud_df = final_df[["Lat", "Cloud"]]
wind_df = final_df[["Lat", "Wind"]]
```


```python
# Lat v Temp scatter plot
plt.figure(figsize=(10,7))
plt.scatter(temp_df["Temp"], temp_df["Lat"], color = "red")
plt.title("Latitude v Current Temperature")
plt.xlabel("Current Temperature (Farenheit)")
plt.ylabel("Latitude")
plt.ylim(-95, 95)
plt.figtext(.95, .5, "Note: \nHorizantal line denotes the Equator \nVertical line denotes the freezing point", fontsize=12)
plt.axhline(y=0, linestyle='--', linewidth = 2, color = "black", alpha=.25)
plt.axvline(x=32, linestyle = "--", linewidth = 1, color = "black", alpha = .25)
plt.savefig("Figures/Lat_Temp.png", bbox_inches = 'tight')
plt.show()
```


![png](output_8_0.png)



```python
# Lat v Hum scatter plot
plt.figure(figsize=(8,7))
plt.scatter(hum_df["Hum"], hum_df["Lat"], color = "red")
plt.title("Latitude v Humidity")
plt.xlabel("Current Humidity (%)")
plt.ylabel("Latitude")
plt.axhline(y=0, linestyle='--', linewidth = 2, color = "black", alpha=.25)
plt.figtext(.95, .5, "Note: \nHorizantal line denotes the Equator", fontsize=12)
plt.savefig("Figures/Lat_Hum.png", bbox_inches = 'tight')
plt.show()
```


![png](output_9_0.png)



```python
plt.figure(figsize=(8,7))
plt.scatter(cloud_df["Cloud"], cloud_df["Lat"], color = "red")
plt.title("Latitude v Cloud Coverage")
plt.xlabel("Current Cloud Coverage (%)")
plt.ylabel("Latitude")
plt.axhline(y=0, linestyle='--', linewidth = 2, color = "black", alpha=.25)
plt.figtext(.95, .5, "Note: \nHorizantal line denotes the Equator", fontsize=12)
plt.savefig("Figures/Lat_Cloud.png", bbox_inches = 'tight')
plt.show()
```


![png](output_10_0.png)



```python
plt.figure(figsize=(8,7))
plt.scatter(wind_df["Wind"], wind_df["Lat"], color = "red")
plt.title("Latitude v Wind Speed")
plt.xlabel("Current Wind Speed (MPH)")
plt.ylabel("Latitude")
plt.axhline(y=0, linestyle='--', linewidth = 2, color = "black", alpha=.25)
plt.figtext(.95, .5, "Note: \nHorizantal line denotes the Equator", fontsize=12)
plt.savefig("Figures/Lat_Wind.png", bbox_inches = 'tight')
plt.show()
```


![png](output_11_0.png)

