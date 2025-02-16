# RefreshControl과 tanstack-query를 함께 사용할 경우 컨텐츠가 튀어오름

## 문제
tanstack-query(react-query), redux-toolkit-query 등 외부 상태 관리 라이브러리에서 제공하는 페칭 상태 값을 사용해 
`RefreshControl`을 제어할 경우 `ScrollView`(`FlatList`, `SectionList` 등)의 컨텐츠가 튀어오르는 현상이 발생함. 

## 해결책
`query.isRefetching`과 같은 값을 직접 사용하지 말고 새로고침 상태를 보관할 수 있는 별개의 상태를 만들어 사용한다.

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

또는 적절한 후크를 작성하여 재사용을 꾀할 수 있다.

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

## 참고
* https://github.com/facebook/react-native/issues/32836
