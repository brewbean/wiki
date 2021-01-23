# wiki
Org wiki

# Table of Contents
[URQL](#urql)
  * [How does caching work?](how-does-caching-work)



## urql

- How does caching work?
  - Important thing to know: _**document caching**_ is a term they made up. It is a reference to how browsers cache _**documents**_, aka web pages (e.g. _index.html_), so that in a traditionaly multipage site, navigating back and forth with your browser history do not require you to request another copy of _about.html_ for example.
  - [Client-Side GraphQL Using URQL (with Phil Pluckthun & Jovi De Croock) — Learn With Jason](https://youtu.be/MYHYv9IxllU?t=2280)
  - Live code-along with urql founders, brief demonstration of how _**document caching**_ works
  - **What does this all mean?** Document based caching as implemented by urql is significantly less sophisticated than my initial assumption. Assume that no caching is done if you are querying a list of things and then querying an item that should be in that list. If you see the use case of the upvote in the video, you can see that document caching is very convenient, but also can overfetch - aka aggressively invalidate cache.

- To make GQL work
  - download GraphQL graphql.vscode-graphql
  - make sure u r on the latest version XD

- When to do GraphQL queries on the component level or page level
  - Pick lazy vs early loading
  - Determine if the user WILL see all of the queries, or if they MIGHT click into one (loading 1 item vs 30 things at once)
  - Pick your battle, both decisions will have their trade offs

## Notes on useMemo and useEffect
`useMemo` and `useEffect` uses the dependency array to trigger a new evaluation on dependency change

```
useEffect(() => {
  console.log('CHANGED')
}, [item])
setItem(2)
// expect 'CHANGED'
setItem(3)
// expect 'CHANGED'
```

`useMemo` we expect to re-evaluate a value on change

```
let x = useMemo(() => a + b, [a, b]) // a = 2 b = 3 on init
```

`x` is then 5

```
// then if a changes
setA(5)
// x gets re-evaluated
// x now 8
```

`useMemo` **should not be necessary for code to work**, it’s a performance optimization.
However, in some advanced cases it is necessary because it keeps a reference value stable over re-renders.

`useCallback` is like `useMemo` except for functions. It is mainly there for keeping a stable reference to a function (component functions get reinit on every render cycle of a component)

Here’s a memoization use case of `useCallback` (uncontroversial use case):
```
let [a, setA] = useState(0)
let [b, setB] = useState(0)
let add = useCallback(() => a + b, [a, b])
add() // 0
setA(5)
add() // 5
add() // using memoized result: 5 (did not need to recompute hence perf. gain)
```
Controversial use case (what I did)
```
let add = useCallback((a, b) => a + b, [])
// this is fundamentally the same as 
// let add = (a, b) => a + b
```
While they are fundamentally the same in general algorithm, they are different in how react initializes the variable on rerender. 

`add` will get recreated if you don’t use `useCallback`. This is inconsequential almost all the time. However, if you’re function is a dependency in a `useEffect` dependency array. 

That’s where there is a problem the problem is that anytime you call `add` in that `useEffect` you probably are triggering a re-render and thus it triggers an infinite loop since `useEffect` is experiencing a change in add (not by the call, but by the re-init).

`useCallback` prevents any re-init of the function and instead preserves its reference.

The final question that might lead to is: Why do we have to include a function in a `useEffect` dependency array?

The answer is because that’s what the lint will complain about which is due to the `exhaustive deps` rule which catches potential memory leaks from any dependency arrays
`exhaustive deps` is a good guideline to have, but with all the caveats, it often leads to confusing errors. One of which is forcing you to include your functions as a dependency, but then seeing infinite calls as a result. The naive solution is to remove your function from the dependency array, but that doesn’t satisfy the lint and didn’t inherently fix the memory leak potential. Using `useCallback` solves both of these issues.

### What kind of scenarios smell like it needs to use `useCallback`?
This is nice when you want no side-effects or not to hook into any lifecycle stuff **remember that all code should work without `useCallback` or `useMemo`**

### When to use Context/Provider?
**If you want to use a hook to pass data around** *it’s not recommended* because context should only be used if multiple components need this data not for code separation (what useMain is doing) for code separation, just separate into a smart container component and dumb component or use this hook pattern (which is buggy, so useCallback controversial use case is that it forces your function to keep track of scoped variables that are changing

### How do i find out if something has been a cache call in the network tab?
if you render a component that uses a `useQuery` and it doesn’t trigger a new network call

###  is it better to do a `useQuery` to grab a single instead of passing down existing data?
The most optimal and least code is to pass down the existing data; however, if we plan on displaying the single item as a widget-like component where you can reuse it anywhere, then the 2nd `useQuery` will make the code simpler and can have the same performance, provided the caching system work. If the info is cached, you can use another query. Caching means that no matter how many `useQuery` s you use, there isn’t a performance penalty
Assuming all your useQuerys hit the cache

The main reason why it is nice to use `useQuery` often and rely on caching is because it simplifies your code. Right now you might not care that your logic is coupled with the list via the prop pass down, but in the future you may want to reuse the single recipe component. By using a separate `useQuery` you guarantee that your code is decoupled. If caching works, then you get free decoupling with no performance hit
If a component is reused in a different context - another page that wasn’t initially planned to use it - then you have to make sure the props are properly plugged in, which can sometimes mean the new page container has to then make a new `useQuery` call to pass in the data for the component. It would be simpler to just have used that `useQuery` and not rely on props in the first place

### Why is it bad to use a hook to store the state & update the state?
The trade off is now having to manage 2 central data stores in terms of scalability though as your logic gets more complex, the container hook method (brain) can help, but it introduces horrible obscure bugs when involving side effects, which is why I rather opt for one app-level context solution rather than smaller context solutions. App-level contexts for storing data would be urql cache or redux



