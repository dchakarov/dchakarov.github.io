---
title: Building a Maze Generation Framework
date: 2025-11-09 00:00:01 Z
layout: post
image: maze.jpg
description: An iOS developer's journey from translating algorithms to building a game foundation
---

<span class="dropcap">I</span>wasn't looking for a game idea when I picked up **Mazes for Programmers** by Jamis Buck. I was just curious about algorithms and looking for something different from the usual data structures textbooks. And the book gave me that - it was incredibly well-written. I couldn't stop reading. But there was one problem: all the code examples were in Ruby.

Ruby is not exactly my cup of tea. As an iOS developer, I am familiar with it, but I never got fluent. But I was itching for a new project, and I wanted to truly understand these algorithms, not just copy code in a language I didn't know. So I embraced the challenge and started converting the code to Swift. What began as a simple translation exercise eventually evolved into a full Swift framework, then a demo app, and ultimately became an iOS game called MZ: Maze Adventures.

This is the story of that journey - specifically, the technical foundation that made everything else possible.

---

## What Makes Maze Algorithms Fascinating

Before diving into the code, let me share what captured my imagination about maze algorithms. They don't just generate mazes; they generate **perfect mazes**. In graph theory terms, a perfect maze is a spanning tree of all cells in the grid. This means:

- The maze is connected (you can reach any cell from any other cell)
- It's acyclic (no loops)
- It has exactly V-1 edges for V vertices
- Most importantly: there's exactly one path between any two cells

But here's where it gets interesting: different algorithms produce mazes with vastly different characteristics, almost like different "personalities."

### Three Algorithms, Three Personalities

**Recursive Backtracker (Depth-First Search)**  
This algorithm explores as far as possible before backtracking, creating long, winding corridors with fewer branches. The result is what's called a "river pattern": paths that players can get trapped in for extended exploration. It has high complexity and is ideal for more challenging puzzles.

**Prim's Algorithm (Randomised)**  
Prim's randomly expands from the growing maze, creating many short dead ends with a tree-like structure. It feels more organic and natural, offering frequent decision points for the player. The complexity is medium, with many branching choices.

**Binary Tree**  
Each cell makes a binary choice (typically north or east), resulting in a strong diagonal bias and predictable texture. Its main benefit? It's very fast to generate. The downside is that the bias makes solutions easier to find. You can often solve these mazes by following the long corridors on the top or right edges.

For MZ: Maze Adventures, I ultimately chose Recursive Backtracker because I liked how the long corridors looked, and the higher complexity made it more challenging for players.

---

## The Ruby to Swift Translation Challenge

When I began translating the algorithms, I had to consider every detail carefully. Ruby and Swift are fundamentally different languages with different paradigms, and that meant more than just syntax changes.

Here's a side-by-side comparison of the Recursive Backtracker algorithm:

**Ruby (from the book):**
```ruby
class RecursiveBacktracker
  def self.on(grid, start_at: grid.random_cell)
    stack = []
    stack.push start_at
    while stack.any?
      current = stack.last
      neighbors = current.neighbors
        .select { |n| n.links.empty? }
      if neighbors.empty?
        stack.pop
      else
        neighbor = neighbors.sample
        current.link(neighbor)
        stack.push(neighbor)
      end
    end
    grid
  end
end
```

**Swift (my framework):**
```swift
public class RecursiveBacktrackerMazeGenerator: MazeGenerating {
    private var stack: [Cell] = []
    
    public func generateStep() 
      -> (generated: [Cell], evaluating: [Cell])? {
        
        let current = stack.last!
        let unvisitedNeighbours = grid
            .neighbours(of: current)
            .filter { $0.links.isEmpty }
        
        if unvisitedNeighbours.isEmpty {
            _ = stack.popLast()
        } else {
            let neighbour = 
              unvisitedNeighbours.randomElement()!
            grid.link(cell1: current, cell2: neighbour)
            stack.append(neighbour)
        }
        return (generated: [current], 
                evaluating: stack)
    }
}
```

The logic is identical - both use a stack to track visited cells, both explore neighbours, and both backtrack when stuck. But the differences reveal each language's philosophy:

- Ruby's `neighbors.sample` is elegant and expressive
- Swift requires `randomElement()!` with explicit unwrapping
- Ruby uses class methods (`def self.on`), while I used instance methods
- Swift's type safety means more explicit handling of optionals

### Beyond One-to-One Translation

Initially, I translated the algorithms one-to-one. But then I wanted to visualise the generation of mazes step by step, not all at once. This forced me to restructure the code - instead of generating a complete maze in one pass, I needed a `generateStep()` method that returned intermediate states.

For three of the twelve algorithms I translated, I abandoned the Ruby code completely and implemented them based purely on the algorithm descriptions in the book. By that point, I understood the patterns well enough that it was actually easier than getting stuck in translation.

---

## Building a Framework, Not Just Code

The real insight came when I decided to build this as a proper Swift framework rather than just a collection of files. This decision paid massive dividends later.

### Protocol-Oriented Design

At the core of the framework is the `MazeGenerating` protocol:

```swift
protocol MazeGenerating {
    var grid: Grid { get set }
    func generateMaze(in grid: Grid)
    func generateStep() -> (generated: [Cell], evaluating: [Cell])?
}
```

This simple protocol allowed for easy swapping of algorithms. Need a different maze style? Just change the algorithm. Want to compare algorithms visually? Iterate through them. The flexibility was incredible.

### Key Design Principles

**Principle 1: Separation of Concerns**
- `Grid` handles cell structure and topology
- `Cell` manages individual cell state and links
- Algorithms implement `MazeGenerating`
- Solvers (like Dijkstra's) work on any valid maze

**Principle 2: Type Safety Prevents Errors**
- Can't link cells from different grids
- Can't access out-of-bounds cells
- Can't create invalid maze states
- Swift's type system catches these at compile time

**Principle 3: Reference Semantics with Observable State for Real-Time Visualisation**

```swift
@Observable
final class Grid {
    private var cells: [[Cell]]
    
    func link(cell1: Cell, cell2: Cell) {
        cell1.links.insert(cell2)
        cell2.links.insert(cell1)
        // Changes automatically notify SwiftUI views
    }
}

@Observable
final class Cell {
    var links: Set<Cell> = []
    var visited: Bool = false
}
```

Why this design choice:
- Grid and cells are classes because they're mutated during maze generation
- The algorithm modifies the grid in place; changes happen instantly
- `@Observable` allows SwiftUI views to automatically redraw as cells are linked
- In the demo app, watching the maze generate step-by-step in real time was only possible because SwiftUI could react to these observable state changes
- The visualisation loop becomes trivial: algorithm mutates observable grid → SwiftUI redraws → user sees generation in progress

The power of this approach: Separating the generative algorithm from UI concerns through `@Observable` meant the demo app didn't need complex bindings or manual refresh logic. SwiftUI's reactive framework automatically handled the visualisation.

---

## From Framework to Demo App

In order to see what I was generating - and this will sound very familiar to iOS developers - I added a demo app to the framework. The demo app was written in SwiftUI and imported the framework.

The demo app lets you:
- Select different maze generation algorithms
- Generate mazes with a button press
- Visualise the maze structure

Then, I added a solver using Dijkstra’s algorithm to show the optimal paths through the maze. Watching the solver trace the perfect path through a Recursive Backtracker maze was mesmerising.

### The "Aha" Moment

Even before finishing the book, I was already thinking: **This could be a game, right?**

Once I could visualise these mazes and watch the solver find optimal paths, the game idea crystallised. Players could attempt to solve mazes with a limited number of moves. The solver could calculate fair move limits. Power-ups could let players bend the rules.

The framework was no longer just a learning project. It was the foundation for something playable.

---

## Lessons Learned

### Framework Separation Pays Off

Building a separate framework compelled me to consider interfaces and contracts. When I later built the game, I could import the framework and immediately have working maze generation - no copy-pasting, no coupling. The framework could be tested independently, and game logic stayed cleanly separated from maze generation.

### Translation Forces Deep Understanding

Translating the Ruby algorithms to Swift made me understand them far more deeply than if I'd just read the Ruby code. I had to think about data structures, error handling, performance, and architecture. Every design decision was intentional, not just inherited from the source.

### Visualisation Changes Everything

The demo app wasn't just a nice-to-have; it fundamentally changed how I understood the algorithms. Seeing the mazes generate, watching the solver work, experimenting with parameters - this hands-on interaction revealed insights I would never have gained from code alone.

### Start Simple, Iterate Quickly

The framework started with just three algorithms and basic functionality. But because the design was sound (thank you, protocol-oriented programming), I added eight more algorithms over time. Each took 30-60 minutes to implement with no changes to existing code.

---

## What's Next

This framework became the foundation for MZ: Maze Adventures, an iOS game that's currently in beta testing. In future articles, I'll dive into:

- Building the game layer on top of the framework
- SwiftUI for game development (pros and cons)
- Progression, power-ups, and game economy

But it all started here: with a curious developer, a Ruby book, and the decision to translate algorithms into Swift.

**Want to explore maze generation yourself?** [The framework is open-source on GitHub](https://github.com/swiftyaf/MazeAlgorithms). Start with the Recursive Backtracker - it's the most rewarding to watch in action. Or if you are into Ruby, I can recommend a great book! Find it here: [Mazes for Programmers](https://pragprog.com/titles/jbmaze/mazes-for-programmers/).
