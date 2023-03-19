
### Rust Streams

Vector / Iterator to stream.
````rust
use futures::stream::{self, StreamExt};

let stream = stream::iter(vec![17, 19]);
assert_eq!(vec![17, 19], stream.collect::<Vec<i32>>().await);
````

Experiments with streams and buffering.

I can highly recommended this article:

[Rust Async Streams - Part 1](https://gendignoux.com/blog/2021/04/01/rust-async-streams-futures-part1.html)
[Rust Async Streams - Part 2](https://gendignoux.com/blog/2021/04/08/rust-async-streams-futures-part2.html)


````rust
use std::future::Future;
use std::time::Duration;
use std::vec::IntoIter;
use futures_util::{Stream, StreamExt, stream, stream::FuturesUnordered};
use futures_util::stream::{FuturesOrdered, Iter, Map};
use lazy_static::lazy_static;
use rand::distributions::Uniform;
use rand::prelude::Distribution;
use tokio::time::{sleep, Instant};

lazy_static! {
    static ref START_TIME: Instant = Instant::now();
}

#[tokio::main]
async fn main() {
    println!("--- exec1 ---");
    exec1().await;
    println!("--- exec2 ---");
    exec2().await;
    println!("--- exec3 ---");
    exec3().await;
    println!("--- exec4 ---");
    exec4().await;
    println!("--- exec5 ---");
    exec5().await;
    println!("--- exec6 ---");
    exec6().await;
}


fn create_vec() -> Vec<i32> {
    let mut a = vec![];
    for i in 0..100 {
        a.push(i);
    }
    a
}


/// Sequential
/// -> Results will be ordered
async fn exec1() {
    let a = create_vec();

    for i in a {
        println!("to_future({}) = {}", i, squared(i).await)
    }
}

/// FuturesUnordered
/// -> Results be unordered
async fn exec2() {
    let a = create_vec();

    // Collect all futures
    let f1: FuturesUnordered<_> = a.iter()
        .map(|i| squared(*i))
        .collect();

    // Then execute them
    let results: Vec<i64> = f1.collect().await;
    for r in results {
        println!("results = {}", r)
    }
}

/// FuturesOrdered
/// -> Results will be ordered
async fn exec3() {
    let a = create_vec();

    // Execution possibility 1: FuturesUnordered
    // Collect all futures
    let f1: FuturesOrdered<_> = a.iter()
        .map(|i| squared(*i))
        .collect();

    // Then execute them
    let results: Vec<i64> = f1.collect().await;
    for r in results {
        println!("results = {}", r)
    }
}

/// Streams
/// -> Results will be ordered
/// -> Requests are run sequentially.
async fn exec4() {
    let a = create_vec();

    let stream = make_stream(a);

    // Then execute them
    let results: Vec<i64> = stream.collect().await;
    for r in results {
        println!("results = {}", r)
    }
}

/// Streams, ordered buffering
/// -> Results will be ordered
async fn exec5() {
    let a = create_vec();

    // Notice the change here
    let stream = make_stream_futures(a);

    // Then execute them
    // Always buffer AFTER take
    let results: Vec<i64> = stream.buffered(10).collect().await;
    for r in results {
        println!("results = {}", r)
    }
}

/// Streams, unordered buffering
/// -> Results will be unordered
async fn exec6() {
    let a = create_vec();

    // Notice the change here
    let stream = make_stream_futures(a);

    // Then execute them
    let results: Vec<i64> = stream.buffer_unordered(10).collect().await;
    for r in results {
        println!("results = {}", r)
    }
}

// Creates a stream where the items are values (not futures).
fn make_stream(a: Vec<i32>) -> impl Stream<Item = i64> {
    stream::iter(a).then(|i| squared(i))
}

// Creates a stream where the items are futures.
fn make_stream_futures(a: Vec<i32>) -> impl Stream<Item = impl Future<Output = i64>> {
    stream::iter(a).map(|i| squared(i))
}

async fn squared(i: i32) -> i64 {
    let millis = Uniform::from(0..20).sample(&mut rand::thread_rng());
    println!(
        "[{}] # get_page({}) will complete in {} ms",
        START_TIME.elapsed().as_millis(),
        i,
        millis
    );

    sleep(Duration::from_millis(millis)).await;
    println!(
        "[{}] # get_page({}) completed",
        START_TIME.elapsed().as_millis(),
        i
    );


    (i * i).into()
}

````
