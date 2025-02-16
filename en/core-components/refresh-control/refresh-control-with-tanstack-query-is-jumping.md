# RefreshControl with tanstack-query is jumping

## Issue
When controlling `RefreshControl` using fetch status values provided by external state management libraries, 
including tanstack-query (formerly called react-query) and redux-toolkit-query, 
the content of `ScrollView` (or `FlatList`, `SectionList`, etc.) is jumping.

## Solution
Use your own state that stores the refreshing status instead of using like `query.isRefetching`:

```tsx
function Solution () {
  const { refetch } = useQuery(...)
  const [ isRefreshing, setIsRefreshing ] = useState(false)
  
  function handleRefresh () {
    setIsRefresh(true)
    refetch().then(() => setIsRefreshing(false))
  }
  
  return (
    <ScrollView
      refreshControl={(
        <RefreshControl
          onRefresh={handleRefresh}
          refreshing={isRefreshing}
        />
      )}  
    />
  )
}
```

Or you can write a hook for reuse purpose:

```tsx
function useRefresh (refreshFunctions: (() => Promise<any>)[]) {
  const [ isRefreshing, setIsRefreshing ] = useState(false)
  
  function handleRefresh () {
    setIsRefreshing(true)
    Promise.all(
      refreshFunctions.map((func) => func())
    ).then(() => setIsRefreshing(false))
  }
  
  const refreshControl = useMemo(() => (
    <RefreshControl
      refreshing={isRefreshing}
      onRefresh={handleRefresh}
    />
  ), [ isRefreshing ])
  
  return { isRefreshing, handleRefresh, refreshControl }
}

function Solution () {
  const { refetch } = useQuery(...)
  const { refetch: refetch2 } = useQuery(...)
  const { refreshControl } = useRefresh([ refetch, refetch2 ])
  
  return (
    <ScrollView
      refreshControl={refreshControl}
    />
  )
}
```


## Reference
* https://github.com/facebook/react-native/issues/32836
