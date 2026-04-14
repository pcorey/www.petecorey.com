---
layout: post
title:  "AI Gems: Part 1"
excerpt: "Let's make ABSOLUTELY sure this object has this name field."
author: "Pete Corey"
date:   2026-04-13
tags: ["Javascript", "AI", "Codex"]
related: []
---

I've been using coding agents more lately, and I've felt the need to document some of the gems they've produced. We'll start on the sarcastic end of that spectrum. Behold:

<pre class='language-javascriptDiff'><code class='language-javascriptDiff'>
+let payload = _.extend({}, form, {
+  name: form.name
+});

 let res = await fetch("/api/foo", {
-  body: JSON.stringify({foo: form}),
+  body: JSON.stringify({foo: payload}),
   method: "POST",
 });
</code></pre>

It should be made clear that in the broader program, there is no chance that `form` will be a non-object. This diff resulted from a seemingly simply and straight-forward prompt of "add a name field to the UI and be sure it's passed up when the form contents are POSTed to the server."

Produced by `gpt-5.4` with Codex.
