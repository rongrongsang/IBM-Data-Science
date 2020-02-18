# TSP Travel Route Planning
##1. Introduction
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
