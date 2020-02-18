# TSP Travel Route Planning
## 1. Introduction
When you have some free days for vacation, where is your dream destination? And how do you plan your travelling?

Paris is my dream destination, i would like to spend half an hour on the last night before departure, searching for points of interest (POI), and then mark them on Google Maps to avoid passing them by. After some operations on my map, it looks like the following:
![map](https://github.com/rongrongsang/IBM-Data-Science/blob/master/Paris.PNG "google map")

I want to walk to each place where I marked and feel the charm of the city. **Here comes the question, how can I walk through all the places with the shortest route?**

I searched Google and didn't find the multi-location path planning function I wanted. The closest requirement is Google's "Add destination" feature. However, this function just connects the places you clicked in turn according to the shortest path. But what we want is not to connect sequentially, but to connect the shortest.

The goal is walking to visit more places in shortest path, actually this is  a TSP question. 
> The travelling salesman problem (TSP) asks the following question: "Given a list of cities and the distances between each pair of cities, what is the shortest possible route that visits each city and returns to the origin city?" It is an NP-hard problem in combinatorial optimization.

More details on [Wikipedia](http://https://en.wikipedia.org/wiki/Travelling_salesman_problem).

Inspired by TSP， I came up with the following solution：

1. Identify all the places that you plan to visit of the city, use Python googlemaps API to get the latitudes and longitudes.

1. Use OR-Tools package to solve TSP problem.

1. Determine travel sequence and visualize it by gmaps.

## 2. Data
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

## 3. Methodology and algorithms
### 3.1 Traveling Salesman Problem (TSP)
Back in the days when salesmen traveled door-to-door hawking vacuums and encyclopedias, they had to plan their routes, from house to house or city to city. The shorter the route, the better. Finding the shortest route that visits a set of locations is an exponentially difficult problem: finding the shortest path for 20 locations is much more than twice as hard as 10 locations.

An exhaustive search of all possible paths would be guaranteed to find the shortest, but is computationally intractable for all but small sets of locations. For larger problems, optimization techniques are needed to intelligently search the solution space and find near-optimal solutions.

### 3.2 Solving TSPs with OR-Tools

No solver can find the shortest paths for all problems. Lots of work has gone into techniques for quickly finding near-optimal solutions, and into proving bounds about how closely these techniques approach optimality.

We can solve TSPs using the OR-Tools [vehicle routing library](https://developers.google.com/optimization/reference/constraint_solver/routing/), a collection of algorithms designed especially for TSPs, and more general problems with multiple vehicles. The routing library is an added layer on top of the [constraint programming](https://developers.google.com/optimization/cp/) solver.

### 3.3 Creat the distance matrix

The distance matrix can be created in 2 ways:

1. Calculate the direct distance using latitudes and longitudes
1. Use google distance matrix API to calculate the REAL distance between 2 POIS

The code below creates the distance matrix using google distance matrix API for the problem.

    DistanceMatrixAPI = "https://maps.googleapis.com/maps/api/distancematrix/json?"

    def get_distance(origins, destinations):
    	res = requests.get(DistanceMatrixAPI+ "origins="+origins+"&destinations="+destinations+API_key)
    	try:
        	return res.json()['rows'][0]['elements'][0]['distance']['value']
    	except:
        	return 0 # same place will return 0

    distance_matrix=[]
    for i in places:
    	for j in places:
        	distance_matrix.append(get_distance(i,j))

    distance_matrix = np.reshape(distance_matrix,(len(places),len(places)))
    distance_matrix

The return as below:
>     array([[    0,  2898,  3438,  1818,  3337,  4598,  1187,  5853,  2912,
         2838, 25859],
       [ 3445,     0,  6055,  2241,  6509,  6335,  4360,  3867,  1764,
          948, 44359],
       [ 2997,  5812,     0,  4732,  2405,  7332,  2952,  8767,  5127,
         5752, 24828],
       [ 1667,  1948,  4276,     0,  4688,  5766,  3097,  4904,  2706,
         1889, 27769],
       [ 2750,  5564,  3108,  4485,     0,  5277,  1656, 14895,  5616,
         5505, 25435],
       [ 4381,  6111,  7696,  4984,  5039,     0,  5017,  6421,  6868,
         6051, 32705],
       [ 1437,  4431,  2817,  3352,  1097,  5067,     0,  7386,  4507,
         4372, 26531],
       [ 5514,  4947,  8951,  5100,  7276,  6162,  6669,     0,  5431,
         4615, 37997],
       [ 1678,  2286,  4287,  1862,  4699,  6112,  3108,  5241,     0,
         2227, 27780],
       [ 3382,  1642,  5991,  2855,  6403,  6948,  4812,  3860,  1701,
            0, 44351],
       [25804, 28618, 23640, 27539, 25712, 32986, 25759, 37889, 28670,
        28559,     0]])

The distance matrix is an array whose i, j entry is the distance from location i to location j in meters, where the locations are given in the order below:

 0 . Place de la Concorde,Paris,France 1. Cathédrale Notre-Dame de Paris,Paris,France 2. Eiffel Tower,Paris,France 3. Musée du Louvre,Paris,France 4. Arc de Triomphe,Paris,France 5. Basilique du Sacré Cœur,Paris,France 6. Av. des Champs-Élysées,Paris,France 7. Cimetière du Père Lachaise,Paris,France 8. Pont des Arts,Paris,France 9. Paris Shakespeare & Company,Paris,France 10. Château de Versailles,Paris,France

### 3.4 Create the distance callback

To use the routing solver, we need to provide a distance (or transit) callback: a function that takes any pair of locations and returns the distance between them. The easiest way to do this is using the distance matrix.

The following function creates the callback, distance_callback and registers it with the solver.

    def distance_callback(from_index, to_index):
	    """Returns the distance between the two nodes."""
	    # Convert from routing variable Index to distance matrix NodeIndex.
	    from_node = manager.IndexToNode(from_index)
	    to_node = manager.IndexToNode(to_index)
	    return data['distance_matrix'][from_node][to_node]

### 3.5 Add the solution printer

The function that prints the solution is shown below.

    def print_solution(manager, routing, assignment):
	    """Prints assignment on console."""
	    print('Objective: {} meters'.format(assignment.ObjectiveValue()))
	    index = routing.Start(0)
	    plan_output = 'Route for vehicle 0:\n'
	    route_distance = 0
	    while not routing.IsEnd(index):
	        plan_output += ' {} ->'.format(manager.IndexToNode(index))
	        previous_index = index
	        index = assignment.Value(routing.NextVar(index))
	        route_distance += routing.GetArcCostForVehicle(previous_index, index, 0)
	    plan_output += ' {}\n'.format(manager.IndexToNode(index))
	    print(plan_output)
	    plan_output += 'Route distance: {}meters\n'.format(route_distance)

The function computes the total distance of the optimal vehicle route, and displays the route and its distance.

The objective is the quantity the solver tries to minimize, namely the total cost of travel. In this example, the objective is the same as total travel distance, but this is not always the case. For this reason, it is a good idea to compute the quantity we want to minimize, as is done in the code above, rather than simply printing the objective.

### 3.6 Main fuction

Now, we have everything to create the main function.

First, we create the problem data:

    data = create_data_model()

Next, declare the index manager and the [routing model solver](https://developers.google.com/optimization/reference/constraint_solver/routing/RoutingModel). (The index manager keeps track of the solver's internal variables corresponding to locations.)

    manager = pywrapcp.RoutingIndexManager(len(data['distance_matrix']),data['num_vehicles'], data['depot'])
    routing = pywrapcp.RoutingModel(manager)

After creating the [distance_callback](https://developers.google.com/optimization/routing/tsp#dist_callback1), set the arc cost evaluator to the transit_callback_index (which is the solver's internal reference to the distance callback). The arc cost evaluator defines the cost of travel between any two locations. In this example, we simply set the cost to be the distance between locations, but in general the costs can involve other factors as well.

    routing.SetArcCostEvaluatorOfAllVehicles(transit_callback_index)

Next, specify the search parameters and a heuristic method to find the first solution:

    search_parameters = pywrapcp.DefaultRoutingSearchParameters()
    search_parameters.first_solution_strategy = (
    	routing_enums_pb2.FirstSolutionStrategy.PATH_CHEAPEST_ARC)

The code sets the first solution strategy to PATH_CHEAPEST_ARC, which creates an initial route by repeatedly adding edges with the least weight that don't lead to a previously visited node (other than the depot). For other options, see [First solution strategy](https://developers.google.com/optimization/routing/routing_options#first_sol_options).

Finally, we can run the solver:

    assignment = routing.SolveWithParameters(search_parameters)

And print the solution:

    if assignment:
    	print_solution(manager, routing, assignment)

The return as below:
 
> Objective: 77198 meters

> Route for vehicle 0:

> 0 -> 10 -> 2 -> 4 -> 6 -> 3 -> 9 -> 8 -> 1 -> 7 -> 5 -> 0

## 4. Results

We can obtain the results as below after following the above algorithms.

0 -> 10 -> 2 -> 4 -> 6 -> 3 -> 9 -> 8 -> 1 -> 7 -> 5 -> 0

Start from 0(Place de la Concorde),then to 10(Château de Versailles),then
 to Eiffel Tower,Arc de Triomphe, Av. des Champs-Élysées, Musée du Louvre, Paris Shakespeare & Company, Pont des Arts, Cathédrale Notre-Dame de Paris, Cimetière du Père Lachaise, Basilique du Sacré Cœur,and finally, return to the start place that is Place de la Concorde.(-> 2 -> 4 -> 6 -> 3 -> 9 -> 8 -> 1 -> 7 -> 5 -> 0)

## 5. Discussion

### 5.1 Observations

After observing the results, we can easily find that it's a long way from 
Place de la Concorde to Château de Versailles. It's more than 20 km and it's unreasonable for people to walk to. Except this road section, other places are suitable to walk to because they are all in the downtown of Paris and not far away.

### 5.2 Recommendations

According to the above observations, I have some recommendation as follows:

Consider travel modes in travelling routes planning project. For long distances, we can travel by bus or subways or by driving. And for short distances, we condiser about walking or biking.

We can reassess the index of optimal solution. In this report, our goal is to get the shortest path. After introducing the travel modes, the shortest time may be our ultimate goal. Of course, this require further research.

## 6. Conclusion

It's very helpful for people to plan their travelling routes by this report  when in a downtown city. We treat these traveling routes planning problems as TSPs and we can use OR-Tools to solve them.

If considering more complicated situations such as multiple transportation options, we'll need further research to get the optimal solution.



 







