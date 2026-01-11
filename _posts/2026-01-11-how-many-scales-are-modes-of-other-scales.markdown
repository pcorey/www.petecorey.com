---
layout: post
title:  "How Many Scales Are Modes of Other Scales?"
excerpt: "Now that we know there are 4096 possible musical scales, let's explore how many of those scales are modes of other scales."
author: "Pete Corey"
date:   2026-01-11
tags: ["Elixir", "Glorious Voice Leader", "Music"]
related: []
---

In the last post, we learned that there are `4096` (`2^12`) possible musical scales. Interesting, but how many of those scales are [modes](https://en.wikipedia.org/wiki/Mode_(music)) of other scales?

To back up a bit, what is a mode? Let's consider the C major scale, `C`, `D`, `E`, `F`, `G`, `A`, and `B`. Remember that this scale, and every major scale, is comprised of a tonal center (`C`), and a set of notes `0`, `2`, `4`, `5`, `7`, `9`, and `11` semitones away from that tonal center. If we take this same set of notes, but hear `D` as the tonal center of the scale, we get a new set of intervals in relation to this new center:

```
0 2 3 5 7 9 10
```

This scale is known as "D [Dorian](https://en.wikipedia.org/wiki/Dorian_mode)", and is a mode of C major. A mode of a scale is another scale made up of the same notes, but with a different notes as its tonal center.

A major scale has a mode where each note in the scale is considered its tonal center. Given that the major scale has seven notes, it goes to reason that there are seven modes of the major scale. For our purposes, let's call all seven of these scales a "scale family."

---

Now that we've defined our terms, given there are `4096` possible scales, how many scale families are there?

Let's start by moving the `get_formulas/0` function we wrote [last time](/blog/2025/06/25/scales-as-multipermutations-of-0-and-1-into-twelve-places/) into the `Glorious.Scale` module for easier use:

```
def get_formulas() do
  Glorious.Enum.multipermutations([0, 1], 12)
  |> Enum.map(fn multipermutation ->
    multipermutation
    |> Enum.chunk_by(& &1)
    |> Enum.map(&length/1)
  end)
  |> Enum.map(fn multipermutation ->
    Enum.reduce(multipermutation, [0], fn d, [h | _] = formula ->
      [h + d | formula]
    end)
    |> Enum.reverse()
    |> Enum.drop(-1)
  end)
end
```

Let's start by grabbing those formulas, and then mapping them over a function to compute all of modes for each formula:

```
Glorious.Scale.get_formulas()
|> Enum.map(Glorious.Scale.build_modes/1)
```

`Glorious.Scale.build_modes/1` takes a scale formula, and returns its set of modes (including the given scale formula):

```
def build_modes(formula) do
  for i <- 0..(length(formula) - 1) do
    rotated = Glorious.Enum.rotate(formula, i)

    root = Enum.at(rotated, 0)

    rotated
    |> Enum.map(fn
      interval when interval >= root -> interval - root
      interval -> interval + 12 - root
    end)
  end
  |> Enum.sort()
end
```

We compute the mode by rotating the given formula by some offset between `0` and the number of elements in the formula, and offsetting every interval in the new formula by the new root.

After we've computed the set of scale families for every scale formula, we can group this list of lists by their identity to lump scale families together:

```
...
|> Enum.group_by(& &1)
```

Lastly, we just need to count the number of keys in our resulting map:

```
...
|> Map.keys()
|> length()
```

And we get our result...

```
351
```

It turns out that out of all of our `4096` possible scales, there are only `351` distinct scale families. Interesting!
