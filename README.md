# wiki
Org wiki

## urql

- How does caching work?
  - Important thing to know: _**document caching**_ is a term they made up. It is a reference to how browsers cache _**documents**_, aka web pages (e.g. _index.html_), so that in a traditionaly multipage site, navigating back and forth with your browser history do not require you to request another copy of _about.html_ for example.
  - [Client-Side GraphQL Using URQL (with Phil Pluckthun & Jovi De Croock) — Learn With Jason](https://youtu.be/MYHYv9IxllU?t=2280)
  - Live code-along with urql founders, brief demonstration of how _**document caching**_ works
  - **What does this all mean?** Document based caching as implemented by urql is significantly less sophisticated than my initial assumption. Assume that no caching is done if you are querying a list of things and then querying an item that should be in that list. If you see the use case of the upvote in the video, you can see that document caching is very convenient, but also can overfetch - aka aggressively invalidate cache.
