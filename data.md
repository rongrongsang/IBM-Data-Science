# TSP Travel Route Planning

## 2. Data ##
### 2.1 Data source
Identify 11 places of Paris as points of interest (POI) as follows:

1. Place de la Concorde,Paris,France
1. Cathédrale Notre-Dame de Paris,Paris,France
1. Eiffel Tower,Paris,France
1. Musée du Louvre,Paris,France
1. Arc de Triomphe,Paris,France
1. Basilique du Sacré Cœur,Paris,France
1. Av. des Champs-Élysées,Paris,France
1. Cimetière du Père Lachaise,Paris,France
1. Pont des Arts,Paris,France
1. Paris Shakespeare & Company,Paris,France
1. Château de Versailles,Paris,France

and the start point is **Place de la Concorde**,Paris,France. I would slept in Hôtel Mayfair Paris on the last night before departure and Place de la Concorde is the nearest place.

### 2.2 Data preparation

We can creat a search fuction via [google API](https://developers.google.com/maps/documentation/distance-matrix/start#get-a-key) and get all the latitudes and longitudes of the POIs.

A typical return from google API is like following:


> {'candidates': [{'formatted_address': '75008 Paris, France',
   'geometry': {'location': {'lat': 48.8656331, 'lng': 2.3212357},
    'viewport': {'northeast': {'lat': 48.86765525, 'lng': 2.32344265},
     'southwest': {'lat': 48.86334185000001, 'lng': 2.319150449999999}}},
   'name': 'Place de la Concorde'}],
 'status': 'OK'}
 
This is a JSON data set, we need to split latitudes and longitudes out.

Build up a fuction codes as below:

    def get_lat_lon(address):
	    res = requests.get(PlaceSearch+address+searchfields+API_key)
	    lat = res.json()['candidates'][0]['geometry']['location']['lat']
	    lon = res.json()['candidates'][0]['geometry']['location']['lng']
	    return [lat,lon]
to get the latitudes and longitudes for each POI.

1. Place de la Concorde,Paris,France [48.8656331, 2.3212357]
1. Cathédrale Notre-Dame de Paris,Paris,France [48.85296820000001, 2.3499021]
1. Eiffel Tower,Paris,France [48.8580574, 2.2945162]
1. Musée du Louvre,Paris,France [48.8606111, 2.337644]
1. Arc de Triomphe,Paris,France [48.8737917, 2.2950275]
1. Basilique du Sacré Cœur,Paris,France [48.88670459999999, 2.3431043]
1. Av. des Champs-Élysées,Paris,France [48.8697953, 2.3078204]
1. Cimetière du Père Lachaise,Paris,France [48.861393, 2.3933276]
1. Pont des Arts,Paris,France [48.85834240000001, 2.3375084]
1. Paris Shakespeare & Company,Paris,France [48.852547, 2.3471197]
1. Château de Versailles,Paris,France [48.8048649, 2.1203554]

Then we can mark them in google maps by using [folium](https://pypi.org/project/folium/).

![](https://github.com/rongrongsang/IBM-Data-Science/blob/master/Paris_POI.PNG)












