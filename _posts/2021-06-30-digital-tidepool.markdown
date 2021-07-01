---
layout: post
title:  "Building the Digital Tidepool"
date:   2021-06-30 12:00:00 -0500
categories: coding digital-tidepool projects
published: true
---

# Howdy Folks!

In this post I am going to walk you through some of the more technical problems I ran into and the overall developmental process behind my project titled "The Digital Tidepool". 

If you haven't yet visited the tidepool, you should mess around for a bit [here][tidepool-link]. Additionally, if you're interested in diving into the code, the finished project can be found on my [github page][github].

## The First Attempt

In truth, this is the third time I've tried to create the digital tidepool. My first attempt was a huge success for me at the time but looking back I can see many poor design decisions that I made. It was written entirely in python using the PyGame library which allows for some interesting game development / GUI interactivity. As far as a tool goes, it was far out of my range when I first attempted to create the tidepool. 

My first goal was to model the Game of Life, which I was able to do in surprisingly short time given how much documentation and clones there are out there. It was only after this that I decided to add some additional characters and see where things ended up. At the time, I had just finished my last term of college a semester early. I was in the process of job hunting and even had some promising leads until Covid hit. For a couple weeks the only thing I had to keep busy was annoying my girlfriend and stare at little green squares. 

In college I studied Biology. It's no surprise then, that natural systems would find their way into my code. In fact, modelling natural systems is one of the things that interests me most in software. As such, with four years of cellular, ecological, and evolutionary coursework beat into my mind, I decided to reformat some of that information into code. 

This was the result of my first honest attempt:

<iframe src="https://www.youtube.com/embed/sDO5RWVB3wk"
   width="800" height="400" frameborder="0" allowfullscreen></iframe>

Basically, what's happening above is the same as what is happening in the current version. There are green elements which grow and creep across the grid like a mold or algae. These act as the foundation of the ecosystem and are the prey to the blue "eaters" which seek out and feed on the greens. The red element is a predator or a secondary consumer which seeks out the blue eaters and consumes them when possible. 

There were a few things that I did right here, and a lot that I did wrong. I suppose the journey of a developer is to recognize the "right" and the "wrong" of past projects, right?

The single most redeeming feature of this iteration was the graph displaying the population sizes of each element. This is something I would really like to incorporate into the current version of the tidepool in the future. The population graphs between predator and prey ended up closely matching real life data collected from wildlife surveys.

The largest drawback to me at the time was the lack of accessibility. Building something is great, but if I'm not able to allow laymen to access and interact with the project themselves, it feels a bit like wasted energy. The best I could think of doing at the time was taking a couple demo videos and sending them to close friends. 

As for most of the other features of the project... there was room for improvement. 

But time goes on, college purgatory ends, and projects get archived.

## Let's try that again...

I didn't resume work on the tidepool until about a year later. I decided to re-write the entire project in JavaScript and host it using a Heroku based Flask app. This attempt was... not quite as successful as my first attempt. The best result I could manage was a small canvas with a basic rendering of green producers and blue consumers along with some rudimentary stats on the side. The biggest roadblock I was running into was performance issues. Regardless of initial parameters, around 500 timesteps into the simulation a great deal of lag would appear, and the framerate of the simulation would plummet. Additionally, I had no concept of DOM manipulation and I had it in my mind that any processing had to take place behind the scenes, which involved communication between the JavaScript front end to the Python backend. I ran into more and more roadblocks throughout the course of the project and eventually the attempt was abandoned. 

This time, the single redeeming fact of the project was the accessibility. Anybody with the URL could dive in and experience the project firsthand. This was a huge part of the project that I wanted to *work*. Unfortunately, my skills in HTML, CSS and JavaScript were woefully underdeveloped and the project requirements quickly outgrew my own knowledge base. I couldn’t continue because I couldn't determine *what I did not know*. 

While my code for the second iteration was technically written in JavaScript, it was fully pythonic in its implementation and was built entirely against the grain of JavaScript best practices, which likely attributed to the poor performance of the simulation. And yet, while it might have been a failed attempt, I learned a ton about how JavaScript works, and it acted as a powerful primer for this third iteration. 

## Third times the charm!

I began working on this iteration of the digital tidepool about two weeks prior to me writing this. It's incredible what you can get done when you decide to leave your job working as a lab-rat and commit fully to writing software eight plus hours a day! 

Not only is this project the third iteration of an old idea, but it is my submission for my first major portfolio project for my Software Engineering course. 

There are many things that set this iteration apart from others. For starters, this project is written entirely with native HTML, CSS, and JavaScript, all coded by hand. The only external package utilized was the PIXI library, which is used for editing and interacting with canvas objects and allows for pretty powerful sprite management. 

### Lists of lists of lists

Each time I attempted this project, I had to create a model of the grid. My go to strategy for representing a two-dimensional grid has been to contain a nested list of lists. Nesting lists three levels deep allowed for very easy indexing across the x and y axes, and each x/y pair would return a list corresponding to that index on the board. I would use the indices of the deepest list to represent elements on the board. The first index of the list would identify the space as empty, green, eater or hunter. The subsequent indices would represent age, number of prey eater or any other properties of the element that needed persistence. For example, calling the index of the grid might have returned something like this: 

{% highlight javascript %}

grid[5][5]
//=> [0,0,0]
//Grid space is empty. 

grid[4][6]
//=> [1,34,0]
//Grid space is occupied by a green that has existed for 34 steps and has consumed zero elements.

grid[3][3]
//=> [2,60,37]
//Grid space is occupied by an eater that has existed for 60 timesteps and consumed 37 greens.

{% endhighlight %}

This was my go-to strategy for representing the game board across all three of my iterations. It made things easy to crunch because each tick of the game board I would simply iterate over the entire game board and carry out actions based on the identity of each space returned by the first index. For example:

{% highlight javascript %}

for (let x = 0; x < grid.length; x++){
   for (let y = 0; y < grid[0].length; y++){
      if (grid[x][y][0] === 0 ){
         // Grid index is empty, carry out 
         // functions for empty grid space
      } else if (grid[x][y][0] === 1){
         // Grid index has an ID value of 1, meaning in my case,
         // the grid space is occupied by a green element.
      } else if (grid[x][y][0] === 2){
         // Grid index has an ID value of 2, meaning in our case,
         // the grid space is occupied by a green consumer element.
      }
   }
}

{% endhighlight %}

While this strategy worked, there was certainly room for improvement in terms of efficiency. I'm not an expert on Big O notation but I do know that nested loops are to be avoided when possible. 

### Objectively Better Code 

To break out of this routine of nested looping, I decided to take the plunge into refactoring my project into a class-based system. There were tons of advantages to taking this approach and I'm very glad I decided to take the time to make it work. 

Essentially what I did was offload each element type into their own class system. That way, each time a new element is created, regardless of if it’s a green, an eater or a hunter, the constructor function inside the class immediately appends the new element to a master list of elements in that class. 

This change of speed had the advantage of allowing me to iterate over lists of elements, rather than iterating over the entire board and acting conditionally based on what the identity of the grid space was discovered to be. 

To be able to iterate over a known list of hunter elements and allow them to each calculate their next move, then do the same with each other element, was a much cleaner and simpler way to handle things. It even had the added benefit of allowing me to remove properties from the grid list as all relevant properties of an element would be contained within its object properties. One such property referred to its index and therefor placement on the board. 

All in all, this change of tactics was a huge step forward and greatly improved both the speed of the simulation but also the readability of the underlying code. 

### Persistence

Another big roadblock that I ran into was the issue of persistence between simulations. 

My go to method for resetting the simulation is to simply reload the page and start fresh. This was the simplest and cleanest solution, the only downside being that all changes made by the user in the previous session would be lost - all values would be returned to default and any changes the user made would have to be repeated. I wanted the tidepool to be something that users can interact with and experiment with change, so I decided that it was essential to have continuity between sessions. The issue was that I had no solid idea on how to persist data between reloads of the page. 

I landed on two solid ways to solve this problem:

The first involves downloading and installing the node package "JSON-Server" which is a lightweight program that allows a user to create a local database in JSON format that can be called from your local machine using fetch. This worked very well for my purposes as I was able to make a POST request containing all the parameters the user has customized, save all the relevant information in the database, reload the front end, make a GET request back to the database, and finally override the default parameters with the configuration persisted from the last simulation.

I liked this method because it allows for long term storage of configurations as well as the users personal record of longest held balance in the tidepool. I also didn't like this method because it dug into my desire for accessibility. I don't expect the average user to navigate to my github account, clone my project, install npm, install a dependency they have never heard of, run a JSON server and finally get to experience my project. I feel as though by minimizing the number of steps associated with accessing my work, the more engaged users will be. This led me to my second solution.

My second solution to this problem was the humble cookie. I always vaguely knew what cookies were, but I had never experimented with them until this project. Cookies, as defined by [Tech Terms][cookie], are:
"a small amount of data generated by a website and saved by your web browser. Its purpose is to remember information about you, similar to a preference file created by a software application."

In my case, the small piece of data that my project generates are a stringified JSON object containing the current configuration of user parameters. This cookie gets saved right before the page is reloaded, and once the reload is completed, the cookie is accessed and the configurations are populated with the values from the last session, seamlessly.

Overall, I am very pleased with the ease of access and the continuity that cookies have allowed me in this project. They are a powerful tool and I feel as though I've only scratched the surface of their usage. This approach allows for simple short-term experimentation in the tidepool. Once the browser is closed, the configurations will be lost, so the persistence is not quite as extensive as with utilizing the JSON server, but for the average user it is perfect.

## To Conclude...

The digital tidepool is an idea I've had for a long time, and it has been a cathartic experience to finally feel as though I've done it *right*. There are dozens of other issues and roadblocks that I ran into during the production of this third iteration, but I think that I will leave it here and allow you to jump in. I can't say with much confidence that this will be my last time building the tidepool. I think that this idea has a lot to offer in terms of exploration. At the end of the day however, I am not quite set on being a game developer. The sprite art conceived in this project would be put to shame by any who tried. But still, they are mine and I am proud of this project. I hope you've enjoyed reading and I hope you can take something away from this and plug it into your own work. I'm still not awesome at this blogging exercise so if anybody has some pointers, tips, or tricks, shoot me an email. That goes for questions about the project too!

Thanks for reading,

Trace


![Lovey Eater](/assets/post_images/eater-lovey.png)

[tidepool-link]: https://tracedelange.github.io/digital-tidepool/
[github]: https://github.com/tracedelange/
[cookie]: https://techterms.com/definition/cookie

