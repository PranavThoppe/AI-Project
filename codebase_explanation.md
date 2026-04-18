# Berkeley AI Pac-Man Search Project — Codebase Explanation

> **Course**: AI (Spring 2026)  
> **Project**: Search in Pac-Man  
> **Source**: UC Berkeley CS 188 — Intro to Artificial Intelligence

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture Diagram](#2-architecture-diagram)
3. [File-by-File Breakdown](#3-file-by-file-breakdown)
   - [search.py — Search Algorithms (Student Code)](#31-searchpy--search-algorithms-student-code)
   - [searchAgents.py — Search Problems & Agents](#32-searchagentspy--search-problems--agents)
   - [pacman.py — Game Controller & State](#33-pacmanpy--game-controller--state)
   - [game.py — Core Engine](#34-gamepy--core-engine)
   - [util.py — Data Structures & Utilities](#35-utilpy--data-structures--utilities)
   - [ghostAgents.py — Ghost Behaviors](#36-ghostagentspy--ghost-behaviors)
   - [keyboardAgents.py — Human Player Input](#37-keyboardagentspy--human-player-input)
   - [pacmanAgents.py — Simple Pac-Man Agents](#38-pacmanagentspy--simple-pac-man-agents)
   - [layout.py — Map Loading & Representation](#39-layoutpy--map-loading--representation)
   - [eightpuzzle.py — 8-Puzzle Demo](#310-eightpuzzlepy--8-puzzle-demo)
   - [Autograder & Testing Files](#311-autograder--testing-files)
4. [How the Search Algorithms Work](#4-how-the-search-algorithms-work)
5. [How to Run the Project](#5-how-to-run-the-project)
6. [Key AI Concepts Demonstrated](#6-key-ai-concepts-demonstrated)

---

## 1. Project Overview

This project uses the classic **Pac-Man** game as a testbed for **search algorithms** — a foundational topic in Artificial Intelligence. Pac-Man must navigate mazes, collect food dots, and avoid ghosts. The student's job is to implement search algorithms (DFS, BFS, UCS, A*) that let an AI agent automatically find paths through the maze.

The codebase is structured so that the **game engine** (rendering, rules, state management) is already provided. Students only need to modify:

| File | What to Implement |
|---|---|
| `search.py` | DFS, BFS, Uniform-Cost Search, A* Search |
| `searchAgents.py` | CornersProblem, cornersHeuristic, foodHeuristic, findPathToClosestDot, AnyFoodSearchProblem |

---

## 2. Architecture Diagram

```
┌──────────────────────────────────────────────────────────┐
│                     pacman.py (main)                     │
│  - Parses CLI arguments                                  │
│  - Creates GameState, agents, display                    │
│  - Calls runGames() to start the game loop               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐    ┌──────────────┐    ┌─────────────┐  │
│  │  game.py     │    │  layout.py   │    │  util.py    │  │
│  │  - Agent     │    │  - Layout    │    │  - Stack    │  │
│  │  - Directions│    │  - Reads .lay│    │  - Queue    │  │
│  │  - Grid      │    │    files     │    │  - Priority │  │
│  │  - GameState │    │  - Walls,    │    │    Queue    │  │
│  │    Data      │    │    food, etc │    │  - Counter  │  │
│  │  - Actions   │    └──────────────┘    └─────────────┘  │
│  │  - Game loop │                                        │
│  └─────────────┘                                         │
│                                                          │
│  ┌──────────────────────────────────────────────────────┐ │
│  │              AGENT LAYER                             │ │
│  │  ┌─────────────────┐  ┌────────────────────────────┐ │ │
│  │  │ searchAgents.py  │  │  ghostAgents.py            │ │ │
│  │  │ - SearchAgent    │  │  - RandomGhost             │ │ │
│  │  │ - PositionSearch │  │  - DirectionalGhost        │ │ │
│  │  │   Problem        │  └────────────────────────────┘ │ │
│  │  │ - CornersProblem │  ┌────────────────────────────┐ │ │
│  │  │ - FoodSearch     │  │  keyboardAgents.py         │ │ │
│  │  │   Problem        │  │  - KeyboardAgent           │ │ │
│  │  └─────────────────┘  └────────────────────────────┘ │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌──────────────────────────────────────────────────────┐ │
│  │              SEARCH LAYER (Student Code)             │ │
│  │  search.py                                           │ │
│  │  - depthFirstSearch (DFS)                            │ │
│  │  - breadthFirstSearch (BFS)                          │ │
│  │  - uniformCostSearch (UCS)                           │ │
│  │  - aStarSearch (A*)                                  │ │
│  └──────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

---

## 3. File-by-File Breakdown

### 3.1 `search.py` — Search Algorithms (Student Code)

This is the **most important file** — it contains the search algorithms that students implement.

#### `SearchProblem` (Abstract Class)
Defines the interface that every search problem must implement:

| Method | Purpose |
|---|---|
| `getStartState()` | Returns the initial state |
| `isGoalState(state)` | Returns `True` if `state` is a goal |
| `getSuccessors(state)` | Returns list of `(successor, action, stepCost)` triples |
| `getCostOfActions(actions)` | Returns total cost of a sequence of actions |

#### `genericSearch(problem, i)` — Unified DFS/BFS
A single function handles both DFS and BFS by swapping the data structure:

```python
if i is 1:
    open = util.Stack()      # DFS: Last-In, First-Out
    currentPath = util.Stack()
else:
    open = util.Queue()      # BFS: First-In, First-Out
    currentPath = util.Queue()
```

**Algorithm**:
1. Push start state onto `open`
2. Pop a state; if it's the goal, return the path
3. If not yet visited (`closed` list), expand its successors
4. Each successor is pushed onto `open`, along with the path so far
5. Repeat until the goal is found

#### `depthFirstSearch(problem)` → calls `genericSearch(problem, 1)`
- Uses a **Stack** (LIFO) → explores the deepest node first
- **Not optimal** — finds *a* solution, not necessarily the shortest

#### `breadthFirstSearch(problem)` → calls `genericSearch(problem, 2)`
- Uses a **Queue** (FIFO) → explores the shallowest node first
- **Optimal** for uniform-cost edges — guarantees the shortest path

#### `uniformCostSearch(problem)`
- Calls `genericSearch2(problem, None)` (note: `genericSearch2` appears to reference A* logic with no heuristic)
- Uses a **PriorityQueue** ordered by path cost
- **Optimal** — always expands the least-cost node

#### `aStarSearch(problem, heuristic)`
The most sophisticated algorithm implemented:

```python
open = util.PriorityQueue()      # Priority = g(n) + h(n)
currPath = util.PriorityQueue()
closed = []

open.push(problem.getStartState(), 0)
currState = open.pop()

while not problem.isGoalState(currState):
    if currState not in closed:
        closed.append(currState)
        for successor in problem.getSuccessors(currState):
            pathCost = problem.getCostOfActions(finalPath + [successor[1]])
            if heuristic is not None:
                pathCost += heuristic(successor[0], problem)  # f(n) = g(n) + h(n)
            if successor[0] not in closed:
                open.push(successor[0], pathCost)
                currPath.push(finalPath + [successor[1]], pathCost)
    currState = open.pop()
    finalPath = currPath.pop()
```

- **f(n) = g(n) + h(n)**: total estimated cost = actual cost so far + heuristic estimate
- **Optimal** if the heuristic is admissible (never overestimates) and consistent

#### `nullHeuristic` — trivial heuristic returning `0`
When A* uses this, it behaves identically to UCS.

---

### 3.2 `searchAgents.py` — Search Problems & Agents

This file connects search algorithms to the Pac-Man game by defining **search problems** and **agents**.

#### `SearchAgent` (Provided)
A general-purpose agent that:
1. Takes a search function name (e.g., `depthFirstSearch`) and a problem type (e.g., `PositionSearchProblem`)
2. In `registerInitialState()`: creates the problem, runs the search, stores the action list
3. In `getAction()`: replays the stored actions one at a time

#### `PositionSearchProblem` (Provided)
The simplest search problem — find a path to a specific `(x, y)` position:
- **State**: `(x, y)` coordinate
- **Start**: Pac-Man's current position
- **Goal**: defaults to `(1, 1)`
- **Successors**: non-wall adjacent cells in NSEW directions
- **Cost**: configurable via `costFn`, defaults to 1 per step

#### `StayEastSearchAgent` / `StayWestSearchAgent` (Provided)
Demonstrate how different cost functions change behavior:
- `StayEast`: cost = `0.5^x` → prefers eastern (high x) positions
- `StayWest`: cost = `2^x` → prefers western (low x) positions

#### Heuristics (Provided)

| Heuristic | Formula | Use |
|---|---|---|
| `manhattanHeuristic` | `|x1-x2| + |y1-y2|` | Straight-line grid distance |
| `euclideanHeuristic` | `√((x1-x2)² + (y1-y2)²)` | Straight-line Euclidean distance |

#### `CornersProblem` (Student Code — Incomplete)
Find a path that visits all 4 corners of the maze:
- **State space**: must encode position *and* which corners have been visited
- **Goal test**: all 4 corners visited
- Students need to implement `getStartState()`, `isGoalState()`, `getSuccessors()`

#### `cornersHeuristic` (Student Code — Incomplete)
Must be **admissible** and **consistent**. Currently returns `0` (trivial).

#### `FoodSearchProblem` (Provided)
Find a path that collects *all* food dots:
- **State**: `(pacmanPosition, foodGrid)` where `foodGrid` is a boolean Grid
- **Goal**: `foodGrid.count() == 0` (no food remaining)
- **Successors**: moving to adjacent cells and marking food as eaten

#### `foodHeuristic` (Student Code — Incomplete)
Must be consistent. Currently returns `0`.

#### `ClosestDotSearchAgent` & `AnyFoodSearchProblem` (Student Code — Incomplete)
- `ClosestDotSearchAgent`: greedily finds the nearest food dot repeatedly
- `AnyFoodSearchProblem`: search problem where the goal is *any* food dot

#### `mazeDistance(point1, point2, gameState)` (Provided Utility)
Computes actual maze distance between two points using BFS — useful for heuristics.

---

### 3.3 `pacman.py` — Game Controller & State

The main entry point. Run `python pacman.py` to start the game.

#### `GameState` Class
The central data structure representing the full state of a Pac-Man game:

| Method | Returns |
|---|---|
| `getLegalActions(agentIndex)` | Legal moves for the specified agent |
| `generateSuccessor(agentIndex, action)` | New GameState after the action |
| `getPacmanPosition()` | `(x, y)` tuple |
| `getGhostPositions()` | List of ghost `(x, y)` positions |
| `getFood()` | Grid of boolean food indicators |
| `getWalls()` | Grid of boolean wall indicators |
| `getScore()` | Current score |
| `isWin()` / `isLose()` | Terminal state checks |

#### `PacmanRules` & `GhostRules`
- **PacmanRules**: handles movement, eating food (+10 pts), eating capsules (scares ghosts), winning (+500 pts)
- **GhostRules**: handles ghost movement (can't reverse direction), collision detection, scared ghosts

#### `ClassicGameRules`
Manages game flow — starting, processing turns, win/lose conditions, timeouts.

#### Scoring System
| Event | Points |
|---|---|
| Eat a food dot | +10 |
| Eat all food (win) | +500 |
| Eat a scared ghost | +200 |
| Get caught by ghost | -500 |
| Each time step | -1 |

#### `readCommand(argv)`
Parses command-line arguments:

| Flag | Description | Default |
|---|---|---|
| `-l` / `--layout` | Map layout file | `mediumClassic` |
| `-p` / `--pacman` | Agent type | `KeyboardAgent` |
| `-g` / `--ghosts` | Ghost agent type | `RandomGhost` |
| `-a` / `--agentArgs` | Args passed to agent (e.g., `fn=bfs`) | — |
| `-n` / `--numGames` | Number of games to play | `1` |
| `-q` | Quiet mode (no graphics) | `False` |
| `-z` / `--zoom` | Graphics zoom | `1.0` |

---

### 3.4 `game.py` — Core Engine

The underlying game engine that everything else is built on.

#### `Agent` (Base Class)
Abstract base class — all agents must implement `getAction(state)`.

#### `Directions`
Enum-like class with constants: `NORTH`, `SOUTH`, `EAST`, `WEST`, `STOP`, plus `LEFT`, `RIGHT`, and `REVERSE` mappings for relative direction changes.

#### `Configuration`
Holds an agent's `(x, y)` position and facing direction. `generateSuccessor(vector)` computes a new configuration after moving.

#### `AgentState`
Tracks an agent's full state:
- `configuration` — current position and direction
- `isPacman` — `True` for Pac-Man, `False` for ghosts
- `scaredTimer` — countdown for how long a ghost remains scared

#### `Grid`
A 2D boolean array representing the game board:
- `grid[x][y]` → `True`/`False` (wall/food at that position)
- Supports `copy()`, `count()`, `asList()`, and efficient bit-packing for hashing

#### `Actions`
Static utility methods for movement:
- `directionToVector(direction)` → `(dx, dy)` e.g., `NORTH → (0, 1)`
- `vectorToDirection(vector)` → direction string
- `getPossibleActions(config, walls)` → legal moves from a position

#### `GameStateData`
Internal data container for `GameState` — stores food grid, capsules, agent states, score, and change tracking.

#### `Game`
The main game loop controller:
1. Initializes agents via `registerInitialState()`
2. Loops: solicits actions from each agent in turn, applies them, updates display
3. Handles timeouts and exceptions
4. Records move history for replays

---

### 3.5 `util.py` — Data Structures & Utilities

Provides the data structures used by search algorithms:

#### Core Data Structures

| Class | Type | Used By |
|---|---|---|
| `Stack` | LIFO (list with `append`/`pop`) | DFS |
| `Queue` | FIFO (list with `insert(0,..)`/`pop()`) | BFS |
| `PriorityQueue` | Min-heap via `heapq` | UCS, A* |
| `PriorityQueueWithFunction` | PriorityQueue with auto-priority | Drop-in replacement |

#### `PriorityQueue` Details
```python
def push(self, item, priority):
    entry = (priority, self.count, item)  # count breaks ties
    heapq.heappush(self.heap, entry)

def pop(self):
    (_, _, item) = heapq.heappop(self.heap)
    return item
```
Uses a min-heap — lowest priority is popped first. The `count` field ensures FIFO ordering among equal-priority items.

#### `Counter` (extends `dict`)
A dictionary that defaults missing keys to `0`. Supports:
- Arithmetic: `+`, `-`, `*` (dot product)
- `normalize()` — makes values sum to 1 (probability distribution)
- `argMax()` — key with highest value

#### Other Utilities
- `manhattanDistance(xy1, xy2)` — grid distance
- `nearestPoint(pos)` — snaps to nearest grid cell
- `flipCoin(p)` — returns `True` with probability `p`
- `sample(distribution)` — weighted random sampling
- `TimeoutFunction` — wraps a function with a time limit

---

### 3.6 `ghostAgents.py` — Ghost Behaviors

#### `GhostAgent` (Base)
Chooses actions by sampling from a probability distribution over legal moves.

#### `RandomGhost`
Uniform random — assigns equal probability to all legal actions.

#### `DirectionalGhost`
More realistic ghost AI:
- **Normal mode** (`prob_attack=0.8`): 80% chance to move toward Pac-Man, 20% random
- **Scared mode** (`prob_scaredFlee=0.8`): 80% chance to flee, 20% random
- Distance measured by Manhattan distance to Pac-Man

---

### 3.7 `keyboardAgents.py` — Human Player Input

#### `KeyboardAgent`
Maps keyboard input to game actions:
- `WASD` or arrow keys for movement
- `Q` to stop
- If no key pressed, continues in the last direction

#### `KeyboardAgent2`
Second player controls using `IJKL` keys (for two-player scenarios).

---

### 3.8 `pacmanAgents.py` — Simple Pac-Man Agents

#### `LeftTurnAgent`
Always tries to turn left. If it can't, it goes straight, then right, then reverses. A deterministic, non-search-based agent.

#### `GreedyAgent`
Evaluates all successor states using a scoring function and picks the best immediate move. Uses `scoreEvaluation` (just `state.getScore()`) by default. This is a **local search** — no lookahead.

---

### 3.9 `layout.py` — Map Loading & Representation

#### `Layout` Class
Parses `.lay` text files into game-ready data structures:

| Character | Meaning |
|---|---|
| `%` | Wall |
| `.` | Food dot |
| `o` | Power capsule |
| `P` | Pac-Man start position |
| `G` | Ghost start position |
| `1-4` | Specific ghost numbers |

Example (`tinyMaze.lay`):
```
%%%%%
%   %
% %%% 
%P  .%
%%%%%
```

The `Layout` object stores:
- `walls` — Grid of wall positions
- `food` — Grid of food positions
- `capsules` — list of capsule `(x,y)` coordinates
- `agentPositions` — ordered list of `(isPacman, position)` tuples

**37 layout files** are provided in the `layouts/` directory, ranging from `tinyMaze` to `bigMaze`, plus specialized layouts for corners, search, and classic game modes.

---

### 3.10 `eightpuzzle.py` — 8-Puzzle Demo

A separate search problem demonstrating that the same search algorithms work on **any** problem that implements the `SearchProblem` interface.

#### `EightPuzzleState`
Represents the classic 8-puzzle (3×3 grid with tiles 1–8 and one blank):
```
-------------
| 1 |   | 2 |
-------------
| 3 | 4 | 5 |
-------------
| 6 | 7 | 8 |
-------------
```
- `legalMoves()` → which directions the blank can slide
- `result(move)` → new puzzle state after sliding

#### `EightPuzzleSearchProblem`
Wraps `EightPuzzleState` as a `SearchProblem`:
- **Start state**: the scrambled puzzle
- **Goal state**: tiles in order `[0,1,2,3,4,5,6,7,8]`
- **Successors**: all legal slide moves, each costing 1
- Uses BFS from `search.py` to solve

---

### 3.11 Autograder & Testing Files

| File | Purpose |
|---|---|
| `autograder.py` | Runs all test cases and grades the project |
| `submission_autograder.py` | Submission-specific grading |
| `grading.py` | Grading infrastructure (test tracking, scoring) |
| `testClasses.py` | Base classes for test cases |
| `searchTestClasses.py` | Search-specific test cases (graph problems, heuristic checks) |
| `testParser.py` | Parses `.test` and `.solution` files |
| `projectParams.py` | Project-level parameters |
| `test_cases/` | Directory of individual test case files |

Run the autograder with:
```bash
python autograder.py
```

---

## 4. How the Search Algorithms Work

### Search as a Tree/Graph Traversal

All search algorithms follow the same general pattern:

```
1. Initialize a FRONTIER with the start state
2. Initialize a CLOSED set (visited states)
3. LOOP:
   a. If frontier is empty → FAILURE
   b. Remove a node from the frontier
   c. If it's a goal → return the path
   d. If not in CLOSED:
      - Add to CLOSED
      - Expand: get successors, add to frontier
```

The **only difference** between algorithms is **how the frontier is organized**:

| Algorithm | Frontier | Ordering | Optimal? | Complete? |
|---|---|---|---|---|
| **DFS** | Stack (LIFO) | Deepest first | ❌ No | ✅ Yes (with cycle check) |
| **BFS** | Queue (FIFO) | Shallowest first | ✅ Yes (uniform cost) | ✅ Yes |
| **UCS** | PriorityQueue | Lowest g(n) first | ✅ Yes | ✅ Yes |
| **A*** | PriorityQueue | Lowest g(n)+h(n) first | ✅ Yes (admissible h) | ✅ Yes |

Where:
- **g(n)** = actual cost from start to node n
- **h(n)** = heuristic estimate from n to goal
- **f(n) = g(n) + h(n)** = total estimated cost

### Visual Example

On `mediumMaze`, BFS explores states level by level (like a ripple), while DFS dives deep down one path before backtracking:

```
BFS exploration pattern:      DFS exploration pattern:
    1 1 1                         1
  1 2 2 1                        2
1 2 3 2 1                       3
  1 2 2 1                      4
    1 1 1                     5 (goes deep first)
```

---

## 5. How to Run the Project

### Play Pac-Man manually:
```bash
python pacman.py
```

### Run with a search agent:
```bash
# DFS on mediumMaze
python pacman.py -l mediumMaze -p SearchAgent -a fn=dfs

# BFS on mediumMaze
python pacman.py -l mediumMaze -p SearchAgent -a fn=bfs

# A* with Manhattan heuristic
python pacman.py -l mediumMaze -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic

# UCS
python pacman.py -l mediumMaze -p SearchAgent -a fn=ucs
```

### Run the autograder:
```bash
python autograder.py
python autograder.py -q 1    # Run only question 1
```

### Run the 8-puzzle:
```bash
python eightpuzzle.py
```

---

## 6. Key AI Concepts Demonstrated

### 1. State Space Search
The entire project revolves around formulating problems as state-space searches. A **state** encodes everything needed to describe the current situation (position, food remaining, corners visited, etc.).

### 2. Graph Search vs. Tree Search
The implementations use a `closed` list to avoid revisiting states — this is **graph search**, which prevents infinite loops in cyclic state spaces.

### 3. Admissible & Consistent Heuristics
- **Admissible**: h(n) ≤ actual cost to goal (never overestimates)
- **Consistent**: h(n) ≤ cost(n→n') + h(n') (satisfies triangle inequality)
- Manhattan distance is both admissible and consistent for grid navigation

### 4. Search Problem Abstraction
The `SearchProblem` interface shows how the **same algorithms** work on completely different problems (Pac-Man navigation, corners problem, food collection, 8-puzzle) — as long as they define `getStartState`, `isGoalState`, `getSuccessors`, and `getCostOfActions`.

### 5. Agent Architecture
The project demonstrates the **agent–environment** interaction model:
- **Agent** observes the state, selects an action
- **Environment** applies the action, returns the new state
- This decoupling lets you swap in different agents (keyboard, search-based, greedy) without changing the game engine

### 6. Trade-offs Between Algorithms
- **DFS**: fast, low memory, but finds suboptimal paths
- **BFS**: optimal (uniform cost), but uses more memory
- **UCS**: handles non-uniform costs optimally
- **A***: best of both worlds — optimal *and* efficient with a good heuristic
