---
layout: post
title:  "Generating Musical Scales"
excerpt: "In my journey to rebuild Glorious Voice Leader in Elixir, I need a way to generate correctly spelled musical scales. Here's how I did it."
author: "Pete Corey"
date:   2025-06-22
tags: ["Elixir", "Glorious Voice Leader", "Music"]
related: []
---

Recently I've revived my long-term passion project, [Glorious Voice Leader](https://petecorey.com/blog/tags/#glorious-voice-leader), with the aim of rebuilding the application using Elixir and Phoenix LiveView. As a part of the project, I need a way of generating correctly spelled musical scales. Let’s explore how I approached that task.

For the Glorious Voice Leader use case, I'm particularly interested in generating [major scales](https://en.wikipedia.org/wiki/Major_scale), but with some luck our approach will generalize to other scales as well.

Let’s take a quick tour of the problem domain and define a few terms so we’re all on the same page.

---

By a ["musical scale"](https://en.wikipedia.org/wiki/Scale_(music)), I’m referring to a root note combined with a list of notes that have a specific relationship to that root note. That relationship between the root note and the rest of the notes in the scale define the "type" of the scale.

For example, a root note of `C`, along with the notes `C`, `D`, `E`, `F`, `G`, `A`, and `B` describe a "C major" scale. The root of the scale is `C`, and it’s a major scale because the distance between the root note and the rest of the notes in the scale are `0`, `2`, `4`, `5`, `7`, `9`, and `11` [semitones](https://en.wikipedia.org/wiki/Semitone) respectively.

Similarly, a root note of `G`, along with the notes `G`, `A`, `B`, `C`, `D`, and `E`, `F#` describe a "G major" scale. The root of the scale is `G`, and the distance between the root and the notes in the scale is `0`, `2`, `4`, `5`, `7`, `9`, `11` semitones respectively. Notice that the `F` needed to change to an `F#` to be `11` semitones from the root, `G`.

The "type" of scale built with notes `0`, `2`, `4`, `5`, `7`, `9`, and `11` semitones from the root is a "major" scale. Scales built with notes `0`, `2`, `3`, `5`, `7`, `8`, and `10` semitones from the root are "natural minor" scales. Each different combination of distances defines a different type of scale. Going forward, we’ll call this list of semitone distances the "formula" of the scale.

---

Armed with that knowledge, let’s start writing a function, `Glorious.Scale.get_scale/2` that takes a root note and a scale formula:

```
def get_scale(%Glorious.Note{} = root, formula) when is_list(formula) do
end
```

An important aspect of diatonic scales, or scales with seven notes, is that each note "name" (or letter), must be used exactly once. We’ll add [accidentals](https://en.wikipedia.org/wiki/Accidental_(music)) (`#`/`b`) to alter each note so that it first the given formula.

We’ll call the unaltered list of note names that we’ll use in the scale the "spelling" of the scale:

```
spelling = Glorious.Note.get_notes(:natural)
```

`Glorious.Note` defines all twelve [chromatic notes](https://en.wikipedia.org/wiki/Chromatic_scale) and their [enharmonic equivalents](https://en.wikipedia.org/wiki/Enharmonic_equivalence):

```
@notes [
  %{name: "C", pitch_class: 0, type: :natural},
  %{name: "C#", pitch_class: 1, type: :sharp},
  %{name: "Db", pitch_class: 1, type: :flat},
  %{name: "D", pitch_class: 2, type: :natural},
  %{name: "D#", pitch_class: 3, type: :sharp},
  %{name: "Eb", pitch_class: 3, type: :flat},
  %{name: "E", pitch_class: 4, type: :natural},
  %{name: "F", pitch_class: 5, type: :natural},
  %{name: "F#", pitch_class: 6, type: :sharp},
  %{name: "Gb", pitch_class: 6, type: :flat},
  %{name: "G", pitch_class: 7, type: :natural},
  %{name: "G#", pitch_class: 8, type: :sharp},
  %{name: "Ab", pitch_class: 8, type: :flat},
  %{name: "A", pitch_class: 9, type: :natural},
  %{name: "A#", pitch_class: 10, type: :sharp},
  %{name: "Bb", pitch_class: 10, type: :flat},
  %{name: "B", pitch_class: 11, type: :natural}
]
```

And a function, `Glorious.Note.get_notes/1` that returns a filtered list of those notes:

```
def get_notes(type) when is_atom(type) do
  @notes
  |> Enum.filter(&(&1.type == type))
  |> Enum.map(&struct(__MODULE__, &1))
end
```

A ["pitch class"](https://en.wikipedia.org/wiki/Pitch_class) is a term used to describe the pitch of a note without the specifics of octave and without the ambiguity of enharmonically equivalent note names.

For example, `C#`, `Db`, and `Ebbb` all have a pitch class of `1` and all sound like the same note.

---

The list of natural notes returned by `Glorious.Note.get_notes/1` may not match the desired ordering of note names in the scale we’re generating. We can find the index of our root note and rotate our spelling to place that note first:

```
naturalized_root = Glorious.Note.naturalize(root)
natural_index = Enum.find_index(spelling, &(&1 == naturalized_root))
ordered_spelling = Glorious.Enum.rotate(spelling, natural_index)
```

`Glorious.Note.naturalize/1` un-alters a note and returns the natural note it was derived from:

```
def naturalize(%__MODULE__{type: :natural} = note) do
  note
end

def naturalize(%__MODULE__{name: <<natural::binary-size(1), "#">>, type: :sharp}) do
  new(natural)
end

def naturalize(%__MODULE__{name: <<natural::binary-size(1), "b">>, type: :flat}) do
  new(natural)
end
```

---

Now that we have our `ordered_spelling`, we can zip it together with our scale’s `formula` and compare the starting distance between each note in our `ordered_spelling` against the desired distance described in our `formula`:

```
notes =
  formula
  |> Enum.zip(ordered_spelling)
  |> Enum.map(fn {interval, natural_note} ->
    scale_pitch_class = rem(interval + root.pitch_class, 12)

    Glorious.Note.new(
      natural_note.name <>
        semitones_to_accidentals(scale_pitch_class - natural_note.pitch_class)
    )
  end)
```

The `Glorious.Scale.semitones_to_accidentals/1` function takes a  number of semitones and returns a string of accidentals that describe that distance:

```
defp semitones_to_accidentals(0) do
  ""
end

defp semitones_to_accidentals(diff) when diff < 0 do
  String.duplicate("b", abs(diff))
end

defp semitones_to_accidentals(diff) when diff > 0 do
  String.duplicate("#", diff)
end
```

We use `b` to flatten notes, or to lower their pitch class. Similarly, we use `#` to sharpen notes, or to raise their pitch class.

Once we’ve built up our list of `notes`, the last thing for us to do it to return our resulting scale as a `Glorious.Scale` struct:

```
%__MODULE__{
  formula: formula,
  notes: notes,
  root: root
}
```

---

Let’s test our solution a bit to make sure it’s working as we’d expect:

```
Glorious.Scale.get_scale(Glorious.Note.new("C"), [0, 2, 4, 5, 7, 9, 11])
```

Calling `Glorious.Scale.get_scale/2` with a `G` root and a major scale formula should give us a G major scale:

```
%Glorious.Scale{
  formula: [0, 2, 4, 5, 7, 9, 11],
  notes: [
    %Glorious.Note{name: "G", pitch_class: 7},
    %Glorious.Note{name: "A", pitch_class: 9},
    %Glorious.Note{name: "B", pitch_class: 11},
    %Glorious.Note{name: "C", pitch_class: 0},
    %Glorious.Note{name: "D", pitch_class: 2},
    %Glorious.Note{name: "E", pitch_class: 4},
    %Glorious.Note{name: "F#", pitch_class: 6},
  ],
  root: %Glorious.Note{name: "G", pitch_class: 7}
}
```

And it does!

Similarly, we should be able to generate an F major scale with a flattened note:

```
Glorious.Scale.get_scale(Glorious.Note.new("F"), [0, 2, 4, 5, 7, 9, 11])
```

And we can:

```
%Glorious.Scale{
  formula: [0, 2, 4, 5, 7, 9, 11],
  notes: [
    %Glorious.Note{name: "F", pitch_class: 5},
    %Glorious.Note{name: "G", pitch_class: 7},
    %Glorious.Note{name: "A", pitch_class: 9},
    %Glorious.Note{name: "Bb", pitch_class: 10},
    %Glorious.Note{name: "C", pitch_class: 0},
    %Glorious.Note{name: "D", pitch_class: 2},
    %Glorious.Note{name: "E", pitch_class: 4}
  ],
  root: %Glorious.Note{name: "F", pitch_class: 5}
}
```

Success!

---

Currently, our solution only works with [diatonic scales](https://en.wikipedia.org/wiki/Diatonic_scale). Generating scales with more or less notes requires more human intervention.

If we’re generating a five note scale, like a major pentatonic scale with a formula of `0`, `2`, `4`, `7`, `9`, which note names should we use? To a human musician, it’s obvious that we should use `C`, `D`, `E`, `G`, and `A`, and not `C`, `D`, `Fb`, `G`, and `A`, but our algorithm doesn’t have any insight as to why that’s obvious.

Similarly, when generating an eight note scale, like [Barry Harris’](https://en.wikipedia.org/wiki/Barry_Harris) Sixth Diminished scale, which note name should we duplicate in our spelling? Even human musicians have [disagreements](https://cochranemusic.com/barry-harris-6th-dim-scale-diminished) about this one. Do we duplicate the `G` and have a `G`, `G#`, `A` sequence, or (probably more correctly) do we duplicate the `A` and have a `G`, `Ab`, `A` sequence?

To answer these questions in an unambiguous way, we’ll need to provide our algorithm with a `spelling`:

```
def get_scale(%Glorious.Note{} = root, formula, spelling \\ Glorious.Note.get_notes(:natural))
  when is_list(formula) and is_list(spelling) and length(formula) == length(spelling) do

```

Everything else stays the same, but now we can generate non-diatonic scales:

```
Glorious.Scale.get_scale(Glorious.Note.new("C"), [0, 2, 4, 5, 7, 8, 9, 11], [
  Glorious.Note.new("C"),
  Glorious.Note.new("D"),
  Glorious.Note.new("E"),
  Glorious.Note.new("F"),
  Glorious.Note.new("G"),
  Glorious.Note.new("A"),
  Glorious.Note.new("A"),
  Glorious.Note.new("B")
])
```

```
%Glorious.Scale{
  formula: [0, 2, 4, 5, 7, 8, 9, 11],
  notes: [
    %Glorious.Note{name: "C", pitch_class: 0},
    %Glorious.Note{name: "D", pitch_class: 2},
    %Glorious.Note{name: "E", pitch_class: 4},
    %Glorious.Note{name: "F", pitch_class: 5},
    %Glorious.Note{name: "G", pitch_class: 7},
    %Glorious.Note{name: "Ab", pitch_class: 8},
    %Glorious.Note{name: "A", pitch_class: 9},
    %Glorious.Note{name: "B", pitch_class: 11}
  ],
  root: %Glorious.Note{name: "C", pitch_class: 0}
}
```

Fantastic! Hopefully, Barry would be proud.
