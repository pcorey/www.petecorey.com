---
layout: post
title:  "Painless Google APIs with Elixir and Assent"
excerpt: "Making OAuth protected requests doesn't need to be painful. Instead of depending on towers of external libraries and code generators, just make the requests directly."
author: "Pete Corey"
date:   2026-09-03
tags: ["Elixir", "Assent"]
related: []
---

With the recent [deprecation of the official Elixir Google APIs project](https://github.com/googleapis/elixir-google-api), I've been looking for the next best way to interact with Google APIs from my Elixir application. It turns out that less is more, and with a bit of help from [Assent](https://assent.hexdocs.pm/Assent.html) to manage OAuth tokens, making requests directly to the APIs is the least painful route.

To start, let's configure Assent in our `runtime.exs` to pull Google credentials from our environment and establish a set of scopes we'll need to access our desired APIs:

```
config :core, :strategies,
  google: [
    client_id: System.get_env("GOOGLE_CLIENT_ID"),
    client_secret: System.get_env("GOOGLE_CLIENT_SECRET"),
    strategy: Assent.Strategy.Google,
    authorization_params: [
      scope:
        [
          "email",
          "openid",
          "https://www.googleapis.com/auth/userinfo.profile",
          "https://www.googleapis.com/auth/calendar",
          "https://www.googleapis.com/auth/contacts.readonly"
        ]
        |> Enum.join(" ")
    ]
  ]
```

Be sure to modify the scopes to fit your needs.

Finish configuring Assent by setting up your routes and a controller according to the [usage instructions](https://assent.hexdocs.pm/Assent.html#module-usage). When handling our provider's callback result, we'll only need the returned `access_token` for our uses:

```
{:ok, %{token: %{"access_token" => access_token}}} ->
  conn
  |> put_session(:access_token, access_token)
  |> put_resp_header("location", "/")
  |> send_resp(302, "")
```

Other fields returned by the provider's callback can be very useful, so modify this to fit your needs.

Now that our user's Google access token is stored in our session, we can extract it and use it to make requests to the Google APIs directly. For example, imagine our `/` controller wants to query the user's calendar lists:

```
access_token = get_session(conn, :access_token)

Req.get!("https://www.googleapis.com/calendar/v3/users/me/calendarList",
  auth: {:bearer, access_token}
)
|> Map.get(:body)
|> Map.get("items")
|> Enum.map(fn
  %{"id" => id, "summaryOverride" => summary} ->
    %{
      id: id,
      summary: summary
    }

  %{"id" => id, "summary" => summary} ->
    %{
      id: id,
      summary: summary
    }
end)
```

And with that, we're free to do whatever we'd like with the user's calendar lists.

Obviously this code can be future proofed, hardened, refactored to be more extensible, and generally improved. The point here is to show that interacting with OAuth protected resources doesn't need to be a difficult endeavor. Just get the token and make the requests; no libraries or code generators needed.

Happy interfacing!
