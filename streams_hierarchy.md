### Exploring the structs and traits in futures crate

Stream (futures::stream::Stream)

- trait
- has associated type Item

StreamExt (futures::stream::StreamExt)

- trait
- extends the Stream trait

Iter (futures::stream::Iter)

- Docs: "Stream for the iter function."

iter (futures::stream::iter)

- function
- takes an (normal) Iterator, turns it into a stream which is always ready to yield the next value.

When to use map()?
When to use then()?

What gets called inside map, then closures?

Normal function (synchronous)?
Async function?

