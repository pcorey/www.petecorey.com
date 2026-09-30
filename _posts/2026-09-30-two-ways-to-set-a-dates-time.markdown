---
layout: post
title:  "Two Ways to Set a Date's Time"
excerpt: "Here are two ways to set a DateTime struct's time to a specific time."
author: "Pete Corey"
date:   2026-09-30
tags: ["Elixir"]
related: []
---

Suppose we have a `DateTime`:

```
now = DateTime.utc_now()
```

We want to set the time on our `DateTime` to midnight. Here are two ways to do that. The first sets the time fields on the `DateTime` struct directly:

```
%{ now |
  hour: 0,
  minute: 0,
  second: 0,
  microsecond: {0, 0}
}
```

And the second converts the `DateTime` to a date, stripping off its time information, and then re-assembles the `DateTime` with a new manually constructed `Time` struct:

```
now
|> DateTime.to_date()
|> DateTime.new!(~T[00:00:00])
```

Pick your poison.
