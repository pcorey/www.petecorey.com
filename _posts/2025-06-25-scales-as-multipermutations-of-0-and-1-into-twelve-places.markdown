---
layout: post
title:  "Scales as Multipermutations of 0 and 1 into Twelve Places"
excerpt: "The itch to write a property test for the musical scale generator we built last time led to explore how to generate all possible scales."
author: "Pete Corey"
date:   2025-06-25
tags: ["Elixir", "Glorious Voice Leader", "Music"]
related: []
---

<style>
.example .one {
  fill: #ff005533;
}

.example .two {
  fill: #ff005511;
}

.example text {
  font-family: monospace;
  fill: #ff005555;
}
</style>

After writing the [musical scale generator](/blog/2025/06/22/generating-musical-scales/) from the previous post, I had an itch to more exhaustively test our solution using a property test. In order to write that test, we’ll need to take a detour and explore the world of scales and find a way to generate arbitrary scale formulas to use as test inputs.

For our purposes, we can think of a scale as a root note that defines the ["tonal center"](https://en.wikipedia.org/wiki/Tonality) of the scale, and a collection of notes that have a relationship to that root. For example, a C major scale has a root note of `C`, and six other notes, `D`, `E`, `F`, `G`, `A`, and `B`.

The relationship of each of those notes to the root can be defined by the ["interval"](https://en.wikipedia.org/wiki/Interval_(music)) between them. In the case of the C major scale, those intervals are a major second, a major third, a perfect fourth, a perfect fifth, a major sixth, and a major seventh, respectively. In fact, every major scale follows this pattern. For our purposes, we’ll call this the "formula" of the major scale.

There are other ways of defining this intervalic relationship. Some people define a scale by the intervals between consecutive notes in the scale. For example, our C major scale is built from a series of consecutive whole step, whole step, half step, whole step, whole step, whole step, half step (WWHWWWH) intervals, starting from the root note, `C`:

<svg viewBox="0 0 12 1" class="example">
  <rect x="0" y="0" width="1" height="1" class="one"/>
  <text x="0.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">C</text>
  <rect x="1" y="0" width="1" height="1" class="one"/>
  <rect x="2" y="0" width="1" height="1" class="two"/>
  <text x="2.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">D</text>
  <rect x="3" y="0" width="1" height="1" class="two"/>
  <rect x="4" y="0" width="1" height="1" class="one"/>
  <text x="4.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">E</text>
  <rect x="5" y="0" width="1" height="1" class="two"/>
  <text x="5.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">F</text>
  <rect x="6" y="0" width="1" height="1" class="two"/>
  <rect x="7" y="0" width="1" height="1" class="one"/>
  <text x="7.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">G</text>
  <rect x="8" y="0" width="1" height="1" class="one"/>
  <rect x="9" y="0" width="1" height="1" class="two"/>
  <text x="9.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">A</text>
  <rect x="10" y="0" width="1" height="1" class="two"/>
  <rect x="11" y="0" width="1" height="1" class="one"/>
  <text x="11.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">B</text>
</svg>

For our purposes, we’ll define a scale formula in terms of the interval from the scale note to the root note, measured in semitones. For example, our C major scale can be defined as notes `0`, `2`, `4`, `5`, `7`, `9`, and `11` semitones from the root, `C`.

---

Armed with that knowledge, how can we generate arbitrary scale formulas? It’s really a question of splitting the octave, or twelve semitones, into some number of pieces.

My first thought when approaching this problem was to create a list of twelve elements, and assign either a `0` or `1` to each element of that list. Chunks of consecutive numbers could be considered a note of the scale, and the length of that chunk defines the interval from that note to the next.

As an example, here’s what a major scale would look like, given a list of `[0, 0, 1, 1, 0, 1, 1, 0, 0, 1, 1, 0]`:

<svg viewBox="0 0 12 1" class="example">
  <rect x="0" y="0" width="1" height="1" class="one"/>
  <text x="0.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">0</text>
  <rect x="1" y="0" width="1" height="1" class="one"/>
  <text x="1.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">0</text>
  <rect x="2" y="0" width="1" height="1" class="two"/>
  <text x="2.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">1</text>
  <rect x="3" y="0" width="1" height="1" class="two"/>
  <text x="3.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">1</text>
  <rect x="4" y="0" width="1" height="1" class="one"/>
  <text x="4.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">0</text>
  <rect x="5" y="0" width="1" height="1" class="two"/>
  <text x="5.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">1</text>
  <rect x="6" y="0" width="1" height="1" class="two"/>
  <text x="6.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">1</text>
  <rect x="7" y="0" width="1" height="1" class="one"/>
  <text x="7.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">0</text>
  <rect x="8" y="0" width="1" height="1" class="one"/>
  <text x="8.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">0</text>
  <rect x="9" y="0" width="1" height="1" class="two"/>
  <text x="9.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">1</text>
  <rect x="10" y="0" width="1" height="1" class="two"/>
  <text x="10.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">1</text>
  <rect x="11" y="0" width="1" height="1" class="one"/>
  <text x="11.5" y="0.65" font-size="0.5" font-weight="bold" text-anchor="middle" font-family="monospace">0</text>
</svg>

One way to look at this solution is that we’re generating a ["multipermutation"](https://en.wiktionary.org/wiki/multipermutation), or a permutation that allows repeating elements, into twelve places:

```
Glorious.Enum.multipermutations([0, 1], 12)
```

From there, we’ll want to map over each multipermutation and measure the length of each chunk:

```
|> Enum.map(fn multipermutation ->
  multipermutation
  |> Enum.chunk_by(& &1)
  |> Enum.map(&length/1)
end)
```

As it stand now, this solution will give us a formula defined in terms of intervals between consecutive notes (e.g. WWHWWWH). We can convert it to our root-oriented notation by reducing this list down to a running sum of the consecutive intervals:

```
|> Enum.map(fn multipermutation ->
  Enum.reduce(multipermutation, [0], fn d, [h | _] = formula ->
    [h + d | formula]
  end)
  |> Enum.reverse()
  |> Enum.drop(-1)
end)
```

We add a root interval, `0`, to the start of our formula, and remove the octave interval, `12`, from the end. This solution gives us `4096` (or `2^12`) formulas, which represent every possible scale formula, as we’ve defined them!

---

Throwing our solution into a function, `get_formulas/0`, we can test a few properties of our scale generator code:

```
property "scales are generated correctly" do
  check all formula <- member_of(get_formulas()),
            spelling <-
              list_of(member_of(Glorious.Note.get_naturals()), length: length(formula)),
            root <- member_of(spelling),
            scale <- constant(Glorious.Scale.get_scale(root, formula, spelling)) do
    # Scale should have the correct number of notes:
    assert length(scale.notes) == length(formula)

    # Scale spelling should match the provided spelling:
    assert MapSet.new(spelling) ===
             MapSet.new(Enum.map(scale.notes, &Glorious.Note.naturalize/1))

    # The semitone distance from notes to the root should match the formula:
    assert formula == Enum.map(scale.notes, &rem(&1.pitch_class + (12 - root.pitch_class), 12))
  end
end
```

We use [StreamData](https://hexdocs.pm/stream_data/StreamData.html) to generate a scale `formula`, a `spelling` for that scale (since we’re generating non-diatonic scales as well), and a `root`. From there we make assertions about various properties that should hold for our solution.

After running our test suite, it looks like all of our assertions hold. Success!

---

---

While our solution to the problem of generating scale formulas works, I imagine there are other interesting ways of approaching the problem. We've managed to generate all `4096` "scales", but are there other solutions that give a clearer insight, some semantics, into what we're generating?

Something to think about. Happy exploring!
