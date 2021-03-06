---
title: Searching made easy
state: Published
description: >-
    Search problems are one of those problems that every programmer will encounter
    at least once in his or her life. An important note is that this post will
    explain search problems, but these are not “how to lookup relative hits in
    multiple texts” but more “given a map, what is the best route from A to B”. I
    will discuss search problems in general, Astar and other search algorithms.
date: '2015-01-03T16:31:00+01:00'
tags:
    - tag: tech
    - tag: programming
---

## Problem description

Search problems are quite common, for example. A maze solver, a puzzle solver or it’s most common use: a path finder. All search problems can be described as a tree, where each node represents a “state”, to advance from one state to the next we use what is called a “move”. Some of these states are a goal states, these are what we are looking for and preferably as fast as possible with the shortest cost.

## State space of a maze

Given a random maze with one exit and one entry. Each state consists of the position within this maze. The path from a given state is given by going up in the state tree. The exit is marked as a goal state and a move will return all possible follow-up states from a given state. Immediately it can be seen that if the maze has loops and we expand all states as deep as possible we would end up in an endless loop at some point.

## Traversing the state space uninformed

Now we have a way of representing the problem in an uniform matter, the only thing left is to find is a method to get to the goal state(s).

## Depth first

Let’s begin with the most simple one: Depth first search. This search algorithm expands all states as deep as possible, so if the maze has loops this isn’t the best idea, but it is a start. If we wanted we could build a safe guard into the move function to not allow the occurrence of the same state within a branch of the state space twice, this is referred to as branch closing. The reason depth first search is so easy is because we have to only keep track of one branch at a time, moreover it is a recursive algorithm. Below follows a simple python implementation of depth first search.

```python
def DFS(state):
    if(isGoal(state)): return [state];
    states = move(state)
    for s in states:
        res = DFS(s)
        if(res != None):
            return [state] + res

    return None
```

## Breadth first

Breadth first search is a more memory heavy search algorithm, what it does instead of diving into the depth of our state space we expand horizontal first, this way we search all possible states and this way we avoid being stuck on those loops. More importantly is this will always return the shortest path, this is because instead of deeping out, we expand every possible path one step at a time. Note, again we could close some tree branches based upon reoccurring states, but for simplicity sake this is left out.

```python
def BFS(start):
    queue = [[start]]

    # branch is one branch of the tree, it contains states
    branch = None
    while len(queue) > 0 :
        branch = queue.pop(0)

        if isGoal(branch[-1]): break;

        states = move(s[-1])
        # add all follow-up states to the queue
        queue = queue + [ [branch + s] for s in states ]
    return branch
```

As it can be seen, we need to keep track of every branch we create, this can lead to an immense consumption of resources.

## Just think before you act

Now let’s assume every move has a cost associated with it, this cost is given in the form of a positive number and with this cost we create a weighted graph. Instead of traversing everything let’s try and think which states will likely give us the best result. For these examples the move function will not only return the possible states, but also the cost to get there associated with them.

## Dijkstra

The first thing anybody would think of is, just expand the path with the lowest cost so far. And indeed this is a valid method. This algorithm is called after Edgar Dijkstra who invented and showcased this algorithm in the early days of the computers. Now an important side note, within Dijkstra’s algorithm it is specified that we can’t have reoccurring states in our path, so we will apply that within our example as well.

```python
def dijkstra(start):
    queue = [{"cost": 0, "path": [start]}]

    branch = None
    while len(queue) > 0:
        branch = queue.pop(0)

        if isGoal(branch["path"][-1]): break;

        states, costs = move(branch)
        queue = queue + [ {
                "cost": branch["cost"] + costs[i],
                "path": branch["path"] + states[i]
            } for i in range(len(states))
            if states[i] not in branch["path"] # ignore already present states
        ]
        queue = sorted(queue, key=lambda x: x["cost"])
```

The example above is kept simple, instead of resorting the whole queue we could insert the states on the correct position to keep everything sorted, but this is easier to read. As can be seen we also keep track of the whole path again, but we prioritize the expansion of paths with a lower cost. Some observant readers will see that when every cost is the same, we will have the same behavior as breadth first search.

## A\* the star of search algorithms

For the most search problems we have some sort of measure for how far we are away from a end point. Lets call this our estimate and this can be calculated for every state. In our maze example this can be the absolute distance between us and the exit. If we now prioritize the expansion of states to the states with the lowest cost and estimate, we end up with searching the least possible amount of states and finding the optimal solution first.

```python
def AStar(start):
    queue = [{"cost": 0, "estimate": estimate(start), "path": [start]}]

    branch = None
    while len(queue) > 0:
        branch = queue.pop(0)

        if isGoal(branch["path"][-1]): break;

        states, costs = move(branch)
        queue = queue + [ {
                "cost": branch["cost"] + costs[i],
                "estimate": estimate(branch["path"][-1]),
                "path": branch["path"] + states[i]
            } for i in range(len(states))
            # ignore already present states
            if states[i] not in branch["path"]
        ]
        queue = sorted(queue, key=lambda x: x["cost"] + x["estimate"])

    return branch
```

As can be seen, this is really a small alteration to dijkstra’s algorithm.

## End remarks

And this concludes our tour through search space, the given examples are not ultimate solutions and are only there as a guide to explain the algorithms. Also there are lots more search algorithms, but these are the most used and most essential algorithms, basicly if you can reduce a problem to the state space you have a good way of reaching your goal. On top of that, if you have a good estimate function for your state space you have a really really great way to reach your goal with A*. Most of todays GPS programs run either dijkstra or A* to do the path finding.
