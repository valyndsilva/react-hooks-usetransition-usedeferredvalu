# React 18 Hooks - useTransition vs useDeferredValue

## useTranstion

```
const [isPending, startTransition] = useTransition();
```

Returns a stateful value for the pending state of the transition, and a function to start it.

startTransition lets you mark updates in the provided callback as transitions:

```
startTransition(() => {
  setCount(count + 1);
})
```

isPending indicates when a transition is active to show a pending state:

```
function App() {
  const [isPending, startTransition] = useTransition();
  const [count, setCount] = useState(0);

  function handleClick() {
    startTransition(() => {
      setCount(c => c + 1);
    })
  }

  return (
    <div>
      {isPending && <Spinner />}
      <button onClick={handleClick}>{count}</button>
    </div>
  );
}
```

Note: Updates in a transition yield to more urgent updates such as clicks.
Updates in a transition will not show a fallback for re-suspended content. This allows the user to continue interacting with the current content while rendering the update.

## useDeferredValue

```
const deferredValue = useDeferredValue(value);
```

useDeferredValue accepts a value and returns a new copy of the value that will defer to more urgent updates. If the current render is the result of an urgent update, like user input, React will return the previous value and then render the new value after the urgent render has completed.

This hook is similar to user-space hooks which use debouncing or throttling to defer updates. The benefits to using useDeferredValue is that React will work on the update as soon as other work finishes (instead of waiting for an arbitrary amount of time), and like startTransition, deferred values can suspend without triggering an unexpected fallback for existing content.

### Memoizing deferred children

useDeferredValue only defers the value that you pass to it. If you want to prevent a child component from re-rendering during an urgent update, you must also memoize that component with React.memo or React.useMemo:

```
function Typeahead() {
  const query = useSearchQuery('');
  const deferredQuery = useDeferredValue(query);

  // Memoizing tells React to only re-render when deferredQuery changes,
  // not when query changes.
  const suggestions = useMemo(() =>
    <SearchSuggestions query={deferredQuery} />,
    [deferredQuery]
  );

  return (
    <>
      <SearchInput query={query} />
      <Suspense fallback="Loading results...">
        {suggestions}
      </Suspense>
    </>
  );
}
```

Memoizing the children tells React that it only needs to re-render them when deferredQuery changes and not when query changes. This caveat is not unique to useDeferredValue, and it’s the same pattern you would use with similar hooks that use debouncing or throttling.
