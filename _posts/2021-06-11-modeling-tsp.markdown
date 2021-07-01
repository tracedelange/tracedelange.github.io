---
layout: post
date:   2021-06-10 12:00:00 -0500
title:  "Modeling The Travelling Salesman Problem"
categories: python
published: false
tags: python np-hard
---

# Let's take a trip, shall we?

The [Travelling Salesman Problem][tsp-wiki] is a classic [NP-Hard][nphard-wiki] problem that involves determining the shortest traveling route through a given list of cities. 

I'm not the first to explore this problem by a long shot - but I do think that there is a lot to learn from playing around with it. Mainly, I've found that exploring this problem has given me a much better understanding of some of the strengths and weaknesses of computation. Here I am going to walk through how I modeled this problem. In the future, I'll build upon this to explain how I implemented some interesting solutions to the problem. For starters, let's tackle the issue with brute force until we start running into road blocks.

To define the problem, we need to do two things:
1. Generate a list of N cities which will be represented by a pair of X,Y coordinates
2. Graph each point to visualize our 'map' of the cities

For the first step, a simple function to generate (X,Y) coordinates based on a given number of cities and return a list of those cities will work:

{% highlight python %}
def generate_cities(n_cities):
    coordinates = []
    for i in range(n_cities):
        coordinates.append((np.random.randint(0,100), np.random.randint(0,100)))
    return coordinates
{% endhighlight %}

Using our function we can generate a set of coordinates like so:

{% highlight python %}
coordinates = generate_cities(5)

coordinates
[(8, 46), (88, 32), (77, 42), (40, 61), (14, 54)]
{% endhighlight %}

Graphing the cities is a simple matter of splitting the coordinates into lists of X, Y values and calling a matplotlib plot to get an abstracted birds eye view of our cities:


{% highlight python %}
def graph_cities(coordinates):

    x, y = zip(*[(coordinate[0], coordinates[1]) for coordinate in coordinates])
 
    fig, ax = plt.subplots(1, 1, figsize=(7.5,6))
    ax.plot(x, y, 'o', color='black');
    ax.axis('equal')

    plt.xlim(0,100)
    plt.ylim(0,100)
    
    fig.show()

    return fig, ax

graph_cities(coordinates)
{% endhighlight %}


![Coordinates](/assets/post_images/coordinates.png)

We can think of the graph above as a map. Our ultimate goal is to determine the fastest route that allows us to stop at each once. 

For us humans, it's not that challenging of a problem. We can easily see that the fastest route would be to start at either the left side and work to the right, or start at the right side and work to the left. Keep in mind there is no distinguishment from travlling a route forwards or backwards, these two solutions are equal and therefore are counted as one. 

If we were to define the solution to this problem by hand, we could do so by creating a list of coordinate points that we want to visit in a row:
{% highlight python %}

solution = [(8, 46), (14, 54), (40, 61), (77, 42), (88, 32)]

{% endhighlight %}

We can graph this solution by taking our map from before and drawing lines to connect the cities in the order of appearance in our solution list:

{% highlight python %}
def connectpoints(ax,x1,y1,x2,y2):
    ax.plot([x1,x2],[y1,y2],'r-')

def graph_route(coordinates, route):

    x, y = zip(*[(coordinate[0], coordinates[1]) for coordinate in coordinates])
    
    fig, ax = plt.subplots(1, 1, figsize=(7.5,6))

    ax.plot(x, y, 'o', color='black');

    for i in range(len(route)-1):
        try:
            connectpoints(ax, route[i][0],route[i][1],route[i+1][0],route[i+1][1])
        except:
            continue
    
    plt.xlim(0,100)
    plt.ylim(0,100)

    fig.show()

    return fig, ax

graph_route(coordinates, solution)
{% endhighlight %}



{% highlight python %}
{% endhighlight %}







![Coordinates with Solution](/assets/post_images/coordinates_with_solution.png)




Easy peasy! Humans rock. But we don't write code just so we can solve the problem ourselves, lets automate this process.

Our solution algorithm can be broken down into four steps:

<ol>
<li>Given a list of city coordinates, create a list of each possible permutation of the points</li>
<li>Iterate over the list of permutations and determine if any given route is an inverse of any previous solution, if not, append the route to a list of potential solutions</li>
<li>Iterate over each potential solution and calculate the total distance travelled by the route</li>
<li>Return the route with the shortest total distance, and since we calculated it, return the route with the <b>longest</b> route as well. </li>
</ol>



{% highlight python %}

def get_distance(x1,y1,x2,y2):
    d = round(math.hypot(x2-x1, y2-y1),2)
    return d


def exact_solution(coordinates):
    
    x, y = zip(*[(coordinate[0], coordinates[1]) for coordinate in coordinates])
    
    start = time.process_time()

    #1 Generate a list of all possible combinations of routes
    combs = itertools.permutations(coordinates, len(coordinates))

    #2 Iterate through the combinations and append each uniquie route to 'routes'.
    #Routes that are identical forward and backwards are removed since they are equal in distance. 
    routes = []
    for p in combs:
        if p[0] <= p[-1]:
            routes.append(list(p))


    #3 For each unique route, calculate the total distance of the route
    #by summing the distances between each constituent legs of the route
    for route in routes:
        distance = 0
        for i in range(len(route)):
            try:
                d = get_distance(route[i][0], route[i][1],route[i+1][0], route[i+1][1])
                distance = distance + d
            except:
                route.append(round(distance,2))
                continue

    end = time.process_time()
    path_count = len(routes)
    process_time = str(end-start)
    
    #4 Identify the path of shortest and longest distance
    optimum = min(routes, key = lambda t: t[(len(routes[0])-1)])
    worst = max(routes, key = lambda t: t[(len(routes[0])-1)])
    

        
    print('Processing time to find solution: ' + str(process_time) + " Seconds")
    print('Number of paths generated: ' + str(len(routes)))
    print('Longest solution length: ' + str(worst[(len(worst)-1)]))
    print('Shortest solution length: ' + str(optimum[(len(worst)-1)]))

    graph_route_only(coordinates, optimum)
    plt.title('Optimum Route. Distance: ' + str(optimum[(len(worst)-1)]))
    graph_route_only(coordinates, worst)
    plt.title('Least Optimum Route: ' + str(worst[(len(worst)-1)]))

    return optimum, worst

{% endhighlight %}

There's a lot going on in the above function. The comments tagged 1-4 follow along with our steps above, everything else is extra information for gathering information about the solving process. In essence, this function will take all the time it needs to brute force search through every possible combination of cities, calculate the distance and return the <b>most</b> opimum solution as well as the <b>least</b> optimum solution.

We can see the result of calling this function on our original set of coordinates:

{% highlight python %}

shortest, longest = exact_solution(coordinates)

Processing time to find solution: 0.0008179999999988752 Seconds
Number of paths generated: 60
Longest solution length: 270.55
Shortest solution length: 93.39

{% endhighlight %}


![Solved with best route](/assets/post_images/opt_w_dist.png)
![solved with worst route](/assets/post_images/lest_w_dist.png)
 
For an inital input of 5 cities, it took my computer about a millisecond to determine the fastest and shortest routes from a total of 60 unique paths. 

Now that we have a good show of things, let's scale it up!

Here we generate a map with nine cities:

{% highlight python %}
coordinates = generate_cities(9)

shortest, longest = exact_solution(coordinates)

Processing time to find solution: 3.9375920000000004 Seconds
Number of paths generated: 181440
Longest solution length: 554.9
Shortest solution length: 191.81
{% endhighlight %}

![Solved with best route](/assets/post_images/solved_9.png)
![solved with worst route](/assets/post_images/solved_9_bad.png)

Now things are getting interesting... We only increased our number of cities by four, yet our processing time increased to about four seconds, aka <b> five-thousands times longer</b> than it took to calculate our solution for a five city map. Additionally, the number of potential solutions to calculate increased by a factor of 3024 from 60 routes to 181,440 routes.

Since we're working with permutations, things are getting out of hand pretty quick


<!-- {% highlight python %}

(math.factorial(n_cities))/2

def n_routes(n_cities):
    return (math.factorial(n_cities))/2

n_routes(5)
60

{% endhighlight %}

Sixty solutions isn't bad! It might take us a few minutes to shuffle through all that informatin but our computers can handle that in no time. 

What about 6 cities? What about 8? 10? 12?

Lets try it out:

{% highlight python %}

n_routes(6)
360

n_routes(8)
20160

n_routes(10)
1814400

n_routes(11)
19958400

n_routes(12)
239500800

{% endhighlight %}

If you didn't notice, we've got a problem here. Since were working with factorials, our numbers are getting really big really quick. 

If we want our program to determine the fastest route through 11 different cities, it will have to evaluate almost 20 million solutions. If we want to determine the solution for 12 cities, our program will have to evalute just under 240 million solutions.

We may be running into some roadblocks here if we want to find the solution for a number greater than 12 cities and still be around when our program gets done finding the solution.

But what if we wanted to find solution to a problem with 30 cities? 40? Going the normal method, we might be waiting around awhile for the answer. That's the interesting thing about NP-Hard problems, a problem with only 12 solutions can take a computer 20 minutes to deterine the 100% correct solution. 

Theres a suprisingly simple solution to this problem, and I can't imagine a perfectionist would like the answer. In reality, we dont need the 100% correct answer. If we could get 99% of the way there with 1% uncertainty in one millionth the amount of time, that is a tradeoff worth taking every time. This is idea is sometimes refered to as <b>heuristics</b>, basically the idea that we don't really need a 100% accurate answer. Close enough is good enough. If we can accept this, we can start to get tricky with the travelling salesman problem. 

Now that we've got the problem defined, we'll take a look next time at some of the interesting ways to solve it. If you're interested in the code I used here, you can find the repository [here]:(https://github.com/tracedelange/traveling-salesman-problem)

Thanks for reading, I hope you enjoyed.

Trace -->


[tsp-wiki]:https://en.wikipedia.org/wiki/Travelling_salesman_problem
[nphard-wiki]:https://en.wikipedia.org/wiki/NP-hardness

{% highlight python %}
{% endhighlight %}