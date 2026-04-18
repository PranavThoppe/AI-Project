# CS 4365 Final Project: Search in Pac-Man

This project is a continuation of previous search assignments. While the autograder will test all questions (Q1-Q8), your grade for this final project will be based specifically on **Question 7** and **Question 8**.

**Due Date:** Sunday, April 26, 2026

## Core Tasks

### Question 7: Eating All The Dots (50 points)
Implement a consistent heuristic for the `FoodSearchProblem` to collect all dots efficiently.
- **File to edit:** `searchAgents.py`
- **Function:** `foodHeuristic`
- **Goal:** Find an optimal solution to `trickySearch` while expanding as few nodes as possible.
- **Grading:** Points are awarded based on the number of nodes expanded (fewer than 7000 for full extra credit).

**Test Command:**
```bash
python pacman.py -l trickySearch -p AStarFoodSearchAgent
```

### Question 8: Suboptimal Search (50 points)
Implement an agent that greedily eats the closest dot to find a reasonably good path quickly.
- **File to edit:** `searchAgents.py`
- **Functions:** `findPathToClosestDot` and `AnyFoodSearchProblem.isGoalState`
- **Goal:** Solve `bigSearch` in under a second with a path cost less than 350.
- **Hint:** The quickest way is to implement the goal test for `AnyFoodSearchProblem` and use an appropriate search function.

**Test Command:**
```bash
python pacman.py -l bigSearch -p ClosestDotSearchAgent -z .5
```

---

## Background & Setup

Although you are only implementing Q7 and Q8, you should be familiar with the following files:
- **`search.py`**: Contains the core search algorithms (DFS, BFS, UCS, A*).
- **`searchAgents.py`**: Contains search-based agents and problem definitions.
- **`pacman.py`**: The main game controller.
- **`autograder.py`**: Used to verify your solutions locally.

**Running the Autograder:**
To test Q7 and Q8 specifically:
```bash
python autograder.py -q q7
python autograder.py -q q8
```

## Submission
Submit your modified `search.py` and `searchAgents.py` via eLearning. Do not rename the files or zip them into a subfolder.
