# gil: Greg's Ideal Language

In my experience, programmers love to talk about programming languages:
what they like, what they dislike, what _they_ would do differently.
I'm no different.

Instead of griping about existing programming languages, though,
I'm taking a different approach.
What if there were an _ideal_ language,
one unconstrained by the usual tedious constraints?
What if I could design _my own_ ideal language?

Well, here it is: GIL, Greg's Ideal Language.
This is a thought experiment in language design.

## What this isn't

This is not a working programming language.
There's no code in here, no compiler, no runtime, no standard library.

## A taster

Here's what a small GIL program looks like:

```
import os

func main(args: [string]): int {
    if args.len() < 1 {
        os.stderr.println("usage: hello NAME")
        return 1
    }

    name = args[0]
    println(f"hello {name}")
    return 0
}
```

Take a look at the [introduction to GIL](docs/intro.md)
to learn more.
