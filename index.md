extends: default.liquid
home_page: true
title: First Post
date: 14 January 2016 21:00:30 -0500
---

## Simple Topology API

Simply add to your cargo.toml and you can write your first topology

```toml
[dependencies]
antimony = "0.0.1"
```
---

## Example Topology

Define your topology as a JSON file representing the DAG (Directed Acyclic Graph) in a topology.json file

```json
{
    "name": "exclamation_topology",
    "sm_count": 3,
    "instances_per_sm": 5,
    "topology": [
        {
            "item_type": "Spout",
            "name": "words_spout",
            "module": "spouts::words::start_spout",
            "instance_count": 10
        },
        {
            "item_type": "Bolt",
            "name": "exclaim1",
            "module": "bolts::exclaim::exclaim1",
            "instance_count": 3,
            "input_stream": "word"
        },
        {
            "item_type": "Bolt",
            "name": "exclaim2",
            "module": "bolts::exclaim::exclaim2",
            "instance_count": 2,
            "input_stream": "word2"
        }
    ]
}
```

Create a new lib with cargo and place topology.json as follows

```
.
├── Cargo.lock
├── Cargo.toml
├── src
│   ├── bolts
│   │   ├── mod.rs
│   │   └── exclaim.rs
│   ├── lib.rs
│   └── spouts
│       ├── mod.rs
│       └── words.rs
└── topology.json
```

Implement a spout

```rust
use antimony::components::spout::{Spout, BaseSpout};
use antimony::components::Message;
use antimony::components::ComponentConfig;


struct Words {
    spout: Spout
}

impl BaseSpout for Words {
    fn prepare(&mut self) {}

    fn next_tuple(&mut self) {
        self.spout.emit(Message::tuple("word", "ExampleWord"));
    }
}

pub fn start(args: ComponentConfig) {
    let mut e = Spout::new("Word Spout");
    let g = Generator { spout: e.clone() };
    Spout::start(&mut e, g, args);
}
```

Implement a bolt

```rust
use antimony::components::bolt::{Bolt, BaseBolt};
use antimony::components::Message;
use antimony::components::ComponentConfig;

struct Exclaim1{
        bolt: Bolt
}

impl BaseBolt for Exclaim1{
    fn prepare(&mut self){}

    fn process_tuple(&mut self, tuple: Message){
        let word = tuple.content();
        self.bolt.emit(Message::tuple("word2", format!("{}!", word)));
    }
}

pub fn exclaim1(args: ComponentConfig){
    let mut e = Bolt::new("Exclaim1");
    let g = Exclaim1{bolt: e.clone()};
    Bolt::start(&mut e, g, args);
}


struct Exclaim2{
        bolt: Bolt
}

impl BaseBolt for Exclaim2{
	fn prepare(&mut self){}

    fn process_tuple(&mut self, tuple: Message){
        let word = tuple.content();
        println!("{}!", word);
    }
}

pub fn exclaim2(args: ComponentConfig){
	let mut e = Bolt::new("Exclaim2");
	let g = Exclaim2{bolt: e.clone()};
	Bolt::start(&mut e, g, args);
}
```

