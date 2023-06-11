+++
title = "Building Tetris in Rust"
date = 2023-06-23

[taxonomies]
tags = ["tetris", "project"]
+++

<img src="assets/tetris.webp" />

## Introduction

Recently, I had watched the movie [Tetris](https://en.wikipedia.org/wiki/Tetris_(film)), where it tells the story of how a few parties are in a race for negotiating with the bureaucrats of Soviet Union for a license for Tetris.

Before watching this movie, I did not know that Tetris was that big of a deal back in the days. 

I remember playing it on my Nintendo Gameboy Color when I was still little, but as I could not grasp hold of how to play it, I did not understand how could it be fun. 

Though, I should've probably known, as the game has some pretty impressive achievements today. 

Even after almost 40 years, the game is known by everyone worldwide, and still has an active playerbase. It is available on every platform imaginable, holding the world record for the most ported video game.

Thus, I was interested in building it myself to see what it took for such a simple game to be that timeless.

## What I used

### Rust

As I was learning [Rust](https://www.rust-lang.org/), I felt this was a perfect opportunity to build this in. 

The game had simple rules and gameplay that I felt that wouldn't be too difficult before I dig too deep.

### SDL2

For rendering, I used [SDL2](https://github.com/Rust-SDL2/rust-sdl2), in which there was a crate that provided Rust bindings for it. 

I used SDL2 as I wanted to a simple 2D renderer, I didn't want to use something big like the [bevy](https://bevyengine.org/) as it was too much for my needs and I want to built everything mostly from the ground up.

### Other libs

- rand - Random generation
- anyhow - Easy error handling

## Implementation

### Shapes

To define a shape, I implemented it by having an array of 2D coordinate positions.

For example, to create an I block, we can construct one by doing:

```rs
struct Position(i32, i32);

struct Block {
    position: Position,
    shape: Vec<Position>,
    color: Color
}

let block_i = Block { 
    position: (4, 8),
    shape: vec![
        Position(0, 0),
        Position(1, 0),
        Position(2, 0),
        Position(3, 0),
    ],
    color: Color::RGB(0, 255, 255)
};

// O: empty space
// X: block
//
// Output:
// O O O O
// X X X X
// O O O O
// O O O O
```

The tuple `Position` represent x and y values on a 2D coordinate plane.

So, to define an I block at a horizontal rotation, we can do that by having the x position to be in the range of 0..3 as done above.

The `position` in the `Block` struct defines the world-space position of the block.

We can use this to translate the positions of the shapes from local-space to world-space.

```rs
impl Block {
    fn world_block_positions(&self) -> Vec<Position> {
        self.shape
            .iter()
            .map(|local_pos| Position(block.position.0 + local_pos.0, 
                                      block.position.1 + local_pos.1))
            .collect()
    }
}
```

So, using the same example above where we defined an I block, the world-position would be:

```
(4, 8),
(5, 8),
(6, 8),
(7, 8)
```

### Rotations

To handle rotations, I would have an array of the shapes, I am only supporting up to 4 rotation states. We can then increment and decrement an index to cycle through the rotations.

For example, for 2 rotation states for the I block:

```rs
struct Position(i32, i32);

struct Block {
    position: Position,
    shapes: [Vec<Position>; 4],
    shape_index: usize,
    color: Color
}

let blocks_i = Block { 
    position: (4, 8),
    shapes: [
        vec![
            Position(0, 0),
            Position(1, 0),
            Position(2, 0),
            Position(3, 0),
        ],
        vec![
            Position(2, -1),
            Position(2, 0),
            Position(2, 1),
            Position(2, 2),
        ],
    ],
    shape_index: 0,
    color: Color::RGB(0, 255, 255),
};

// O: empty space
// X: block
//
// Output:
// O O O O | O O X O
// X X X X | O O X O
// O O O O | O O X O
// O O O O | O O X O
```

### Grid

The grid is built out of a 2D array, with the values of an optional Color value.

```rs
struct Grid {
    position: Position,
    cells: Vec<Vec<Option<Color>>
}
```

The position here refers to the grid's world position, and defines the position most top-left position of the grid in world-space.

If the value of a cell is `None`, it means that it is an empty space on the grid.

And then when we lock a block onto the grid, we can use the block's color values to populate the grid at that position.

### Collisions

As we are working with a 2D grid, collisions are relatively simple.

We can deem anything as colliding if they are in the same position in the grid in world-space.

However, since our grid is in local-space, to index it, we have to normalize our positions to local-space.

To do this, we get the difference from the block and grid positions in world-space.

```rs
pub enum Collision {
    None,
    Left,
    Right,
    Top,
    Bottom,
}

impl Grid {
    fn is_colliding(&self, block: &Block) -> Collision {
        for block_position in block.world_block_positions() {
            let x = block_position.0 - self.position.0;
            let y = block_position.1 - self.position.1;
        }

        Collision::None
    }
}
```

Now we have our indexes, we can use it check if at a certain cell position, it has a value, which means it is occupied by a locked block.

To make things simpler, we can define this behaviour as colliding at the bottom position as we assume this to be "stacking", though it doesn't necessarily have to.

```rs
pub enum Collision {
    None,
    Left,
    Right,
    Top,
    Bottom,
}

impl Grid {
    fn is_colliding(&self, block: &Block) -> Collision {
        for block_position in block.world_block_positions() {
            let x = block_position.0 - self.position.0;
            let y = block_position.1 - self.position.1;

            if (self.cells[y as usize][x as usize].is_some()) {
                return Collision::Bottom;
            }
        }

        Collision::None
    }
}
```

Now to check if we are colliding outside the surroundings of the grid, we can just do some bounds checking on the size of the grids.

```rs
pub enum Collision {
    None,
    Left,
    Right,
    Top,
    Bottom,
}

impl Grid {
    fn is_colliding(&self, block: &Block) -> Collision {
        for block_position in block.world_block_positions() {
            let x = block_position.0 - self.position.0;
            let y = block_position.1 - self.position.1;

            if x < 0 {
                return Collision::Left;
            }

            if x >= self.cells[0].len() as i32 {
                return Collision::Right;
            }

            if y < 0 {
                return Collision::Top;
            }

            if y >= self.cells.len() as i32 
                || self.cells[y as usize][x as usize].is_some() {
                return Collision::Bottom;
            }
        }

        Collision::None
    }
}
```

## Shortcomings

At this time of writing, the project still has some outstanding issues that has not been addressed.

Perhaps, in the future these will be fixed. But for now, I think the project is fine as it is.

### Handling collisions with rotations

When rotating blocks, it does not account if the new shape positions will collide with any locked blocks.

As a result, in the event that happens, it will lock the current block in place and replace any position that it may have collided.

### Extra space with rotations

As we are storing each rotation state with its positions, this is effectively storing 4 times the amount of memory for each block that we instantiate.

The alternative that I considered was to use an anchor point and use some math to rotate the positions around that.

Though this may be a more complex solution, but you effectively only have to define your shape once, and you have the rest of the rotations already figured out with that algorithm.

### Some unfinished features

- Hard drop
- Hold block
- Rendering blocks queue

## Conclusion

After playing Tetris a few times and building it, I finally understood why Tetris can be addicting for some.

For such a simple game, it has great replayability, and can be quite thrilling to clear multiple rows consecutively to save yourself as you build up enough height.

I had definitely learned a lot building it, though not entirely perfect, but I would say it is minimal but complete. Ultimately, it was worth it.

If you want to have a look or give it a try, here is a link to the GitHub repository: [https://github.com/dante1130/tetris](https://github.com/dante1130/tetris)

