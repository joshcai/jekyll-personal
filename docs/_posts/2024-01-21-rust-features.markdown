---
layout: post
title:  "Rust Features (AoC 2023)"
date:   2024-01-21 06:13:34 +0000
categories: tech
shortlink: rust-features
---

I've been using [Advent of Code](https://adventofcode.com) for the past few years to practice programming languages - both ones I'm familiar with and to learn new ones. One of the great things about Advent of Code is that it ramps up in difficulty and exposes you slowly to different use cases, which makes learning a new language quite practical.

For 2023, I chose to learn Rust, since I was interested in the benefits of the borrow checker and to see if it would be a langauge I would want to program in later. 

Although the borrow checker / memory safety is likely the main reason a lot of people use the language, I wanted to reflect on some of the features that I really appreciated about the language.

### `match` keyword

I think my favorite feature Rust had was the `match` keyword, I haven't personally worked with a non-functional language that had such a construct - I guess Python 3.10 had recently got this feature (or something similar?), but I haven't used it there yet.

I really enjoyed how concise it made some code, for example, from [day 10](https://github.com/joshcai/advent-of-code/blob/5bb4c799137d4d4996928fa229850ce98fcdf93e/src/2023/day10/src/main.rs#L103):

```rust
let next_dir = match curr_pipe {
    'F' => match dir {
        'U' => 'R',
        'L' => 'D',
        _ => todo!(),
    },
    'J' => match dir {
        'D' => 'L',
        'R' => 'U',
        _ => todo!(),
    },
    'L' => match dir {
        'D' => 'R',
        'L' => 'U',
        _ => todo!(),
    },
    '7' => match dir {
        'R' => 'D',
        'U' => 'L',
        _ => todo!(),
    },
    '-' => dir,
    '|' => dir,
    _ => todo!(),
};
```

Since the `match` construct can return a value, it makes it so much simpler to write a nested match statement.

I also really liked how you can use code in the match cases as well, for example, in [day 16](https://github.com/joshcai/advent-of-code/blob/5bb4c799137d4d4996928fa229850ce98fcdf93e/src/2023/day16/src/main.rs#L37):

```rust
match dir {
    'U' | 'D' => { visit(x + dir_tup.0, y + dir_tup.1, dir, input_vec, seen); },
    'L' | 'R' =>  {
        visit(x, y - 1, 'U', input_vec, seen);
        visit(x, y + 1, 'D', input_vec, seen);
    },
    _ => todo!(),
}
```

### Closure syntax

This is purely a syntatic-sugar thing, but I really liked how easy / concise it was to define a closure. 

For example, in [day 11](https://github.com/joshcai/advent-of-code/blob/5bb4c799137d4d4996928fa229850ce98fcdf93e/src/2023/day11/src/main.rs#L39):

```rust
let y_count = row_vec.clone().into_iter().filter(|s| s < &y).count() as u64; 
```

Being able to define the lambda with essentially only 3 extra characters is quite convenient. I think the only minor downside was that it wasn't possible to define a recursive function using the same syntax. 

### Traits 

As a big proponent for composition over inheritance, I enjoyed Rust's implementation of traits and its exclusion of the inheritance feature. I used Golang for a good chunk of my time at Google, and it felt very familiar to using interfaces, which was nice.

I think the only major difference I can think of was the need to use `Box`, which confused me initially, but makes a lot of sense, and has some benefits to readability as well.

Here's some sample code from [day 20](https://github.com/joshcai/advent-of-code/blob/5bb4c799137d4d4996928fa229850ce98fcdf93e/src/2023/day20/src/main.rs):

```rust
trait Module {
    fn recv(&mut self, name: String, high: bool);
    fn send(&mut self) -> Vec<(String, String, bool)>;
}

struct Broadcaster {
    name: String,
    dest: Vec<String>,
}

impl Module for Broadcaster {
    fn recv(&mut self, _name: String, _high: bool) {}
    fn send(&mut self) -> Vec<(String, String, bool)> {
        let mut res = Vec::new();
        for d in self.dest.iter() {
            res.push((d.clone(), self.name.clone(), false));
        }
        res
    }
}
```

I do like the explicit `impl <trait> for <struct>` construct, which helps understand what trait it is actually implementing. In comparison, I usually need to add comments for what interface a method implements in Golang.

### Unit Testing

I don't think I've worked in another language where the unit tests are commonly defined in the source file themselves, but it was especially handy for Advent of Code, where each day's solution is fairly short, and I almost always want to run the unit tests on the example day multiple times.

For example, this code was from [day 1](https://github.com/joshcai/advent-of-code/blob/5bb4c799137d4d4996928fa229850ce98fcdf93e/src/2023/day01/src/main.rs):

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn part1_example() {
        assert_eq!(part1(include_str!("./example.txt")), 142);
    }

    #[test]
    fn part2_example() {
        assert_eq!(part2(include_str!("./example2.txt")), 281);
    }
}
```

One of my favorite parts of the debugging experience was that in Visual Studio Code, the rust-analyzer extension I used had a `Run Test` button that hovered over each test case. Pressing it would launch a terminal that ran that specific test, and then any compile errors would show up next to the code directly.

### Overall Thoughts

Fighting the borrow checker initially was quite annoying, and there were a few times where I wasn't sure why I wanted to use Rust, but after understanding the basic patterns, it really wasn't too big of an issue.

I think as of now, my major complaint is actually having to explicitly cast all the number types, which causes a ton of verbosity for something that normally doesn't cause me any issues in other languages - I wish there was a compile time option or something like that allowed me to indicate that I would be okay with automatic number type casting.

Overall, I could definitely see myself using Rust in the future, particularly for code that needs memory safety / high performance, e.g. possibly for some game engine code. For other projects, e.g. a backend web server, I would probably use Golang still since I feel more productive overall there. 