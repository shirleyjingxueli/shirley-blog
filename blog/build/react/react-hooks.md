# react-hooks
## State hooks
### useState()
#### declear a variable you can update redirectly
#### ```const [state, setState] = useState(initialState)```
#### setState(nextState)
  - the value you want the state to be.
  - **if you pass a function to nextState, it will be treated as an updater function. It must be pure, the pending state as its only arguments, and should return the next state.** React will put your updater function in a queue and re-render your component.During the next render, React will calculate the next state by applying all of queued updaters to the previous state.**It can be used to update state based on the previous state.**

### useReducer()
#### declear a variable with the update logic inside the reducer
#### ```const [state, dispatch] = useReducer(reducer, initialArg, init?)```
  - reducer(state,action): the reducer function that specifies how the state gets updated. It must be pure, should take the state and action as arguments, and should return the next state. state and action can be of any type.
  - initialArg: The value from whichi the initialState is calculated.
  - init: the initial function should return the initial state. **if it is not specified, the initial state is set to ```initialArg```. Otherwise, the initial state is set to the result of calling init(initialArg)**

## Context Hooks
### useContext()
  #### reads and subscribes to a context
  - you can pass any value via context, including function and object.
  #### ```const value = useContext(SomeContext)```
    - someContext: The context that you have previous created width ```createContext```
    - return the context value for the calling component.

      It is determined as the value passed to closest ```SomeContext.Provider``` above the calling component in the tree. if it doesn't have a value, the value is ```undefined```

      If there is no such provider, then the returned value will be the ```defaultValue``` you have passed to ```createContext```.**the defalut value from ```createContext(defalutValue)``` is only used if there is no matching provider above at all.**

      **The default value never changes. If you want to update context, use it with state as described above.**

      The returned value is always up-to-date. React automatically re-renders components that read some context if it changes.

  #### how to update data
  - declear a state, and pass down state as props
  - declear a state, and pass down state as the defaultValue of ```SomeContext.Provider```

  #### how to override the context
  - you can override the context for a part of the tree by **wrapping that part in a provider with a different value.**

  #### how to optimizing re-renders when passing objects and functions
  - wrap the object creation into useMemo
  - wrap the function with useCallback

## Ref Hooks
### useRef()
#### delear a ref that you can hold any value in it. but most often it is used to hold a DOM.
  - you can reference a value that is not needed for rendering.
#### ```const ref = useRef(initialValue)```
  - initialValue: 
    - initial value of the ref object's current property.
    - it can be a value of any type
    - it is ignored after the initial render.
  - returns:
    - return an object with a single property: current
    - it is set to the initial value you have passed.
    - **on the next renders, useRef will return the same object.**
    - **if you pass the ref object to React as a ```ref``` attribute to a JSX node, React will set its current property.** 
#### How to use
  - store information that not affect the visual output
  - store DOM
    - if your want the parent component manipulate the DOM inside of your component. you can use a combination of useRef to hode the component(such as Input) and ```forwardRef()``` to expose it to the parent component.

### useImpreativeHandle()
#### let you customize a ref exposed by your component
#### ```useImperativeHandle(ref, createHandle, dependencies?)```
  - if you don't expose the entire DOM node, for example two methods: focus and scrollIntoView.

## Effect Hooks
### useEffect()
#### connects a component to an external system
#### feature
  - only run on the client. It doesn't run during server rendering.
#### ```useEffect(setup, dependencies?)```
  - **setup:** the function with your Effect's logic. It may also optionally return a cleanup function.
  - **dependencies:** the list of all reactive values referenced inside of your setup code.**Reactive values include props, state, and all variables and function decleared directly inside your component body.**
  #### when react run cleanup function
    - After every re-render with changed dependencies,react will first run the cleanup function with the old value, and run your setup function with new values
    - After your component is removed from DOM, React will run your cleanup function.
  - **returns:** undefined
#### the difference between no dependency, [], have dependency?
  - **no dependency:** run after every re-render
  - **[]:** only run once
  - **have dependency:** run when the dependencies change  
#### how to use
  - connect to an external system
  - wrapping Effects in custom Hooks
  - controlling a non-React widget
  - fetching data with Effects
  - Specifying reactive dependencies
  - updating state based on previous state from an Effect
  - removing unnecessary object dependencies
  - removing unnecessary function dependencies
  - reading the latest props and state from an Effect
    * use ```useEffectEvent()```
    * Effect Events are not reactive and must always be omitted from dependencies of your Effect.
  - display different content on the sever and the client.  


### useLayoutEffect()
#### fires before the browser repaint to screen. you can measure layout here
#### ```useLayoutEffect(setup, dependencies?)```
#### examples
  - measure the position and size of tooltip. 

### useInsertionEffect()
#### fires before react make changes to DOM. Libraries can change insert dynimac css here.**allows inserting elements into the DOM before any layout effets fire.**
#### feature
  - only run on the client
  - can not update state from inside ```useInsertionEffect```
  - By the time ```useInsertionEffect``` runs, refs are not atteched yet.
  - ```useInsertionEffect``` may run either before or after the DOM has been updated. You shouldn't rely on the DOM being updated at any particular time.
  - unlike other types of Effect, which fire cleanup for every Effect and then setup for every Effect, **```useInsertionEffect``` will fire both cleanup and setup one component at a time. This results in an interleaving of the cleanup and setup functions.**
#### ```useInsertionEffection(setup, dependencies?)```

## Performance Hooks

1. To skip calculations and unnecessary re-render

### useMemo()
#### cache the result of expensive calculation. It let you cache the result of a calculation between re-renders.
#### ```const cachedValue = useMemo(calculateValue, dependencies)```
  - **calculateValue:** The function calculate the value that you want to cache.**it should be pure, shole take no arguments and should return a value of any type.**
    #### when the function will be called?
    - initial render
    - next render: return the same value again if the dependencies have not changeed since the last render.Otherwise,it will call calculateValue, return its result and store it so it can be reused later.
  - **dependencies:** the list of all reactive values referenced inside of the calculateValue.
  - **returns:** calculated value or chached value.
#### How to use
  - skipping expensive recalculations（ex: crate or loop over thousands of objects）. **it won't make the first render faster, it only helps you skip unnecessarry work on updates.**
  - skipping re-rendering of components
  - memoizing a dependency of another hook
  - memoizing a function
#### QA
  [the difference between useMemo and useCallback](#the-difference-between-usecallback-and-usememo)

### useCallback()
#### cache a function defination before passing it down to an optimized component
#### ```const cachedFn = useCallback(fn, dependencies)```
  - **fn:** The function that you want to cache. It can take any arguments and return any values.
  - **dependencies:** the list of all reactive values referenced inside of the fn code.
  - **returns:** cached function or the function you have passed during this render.
#### How to use
  - skipping re-rendering of components
  - updating state from a memoized callback
  - preventing an Effect from firing too often
  - Optimizing a custom Hook  
2. To prioritize rendering

### useTransition()
#### mark a state transtion as non-blocking and allow other update to interrupt it.It lets you update the state without blocking the UI.
#### ```const [isPending, startTransition] = useTransition()```
  - no arguments
  - **returns:** an array with two items. isPending--tell you whether there is a pending transition. startTransition--lets you mark a state update as a transition.

### useDefferedValue()
#### defer updating an non-critical part of UI and let other parts update first
#### ```const deferredValue = useDeferredValue(value)```
  - **value:** The value you want to defer. It can have any type.
  - **returns:**
    * first render: the returned deferred value will be the value you provided.
    * during updates: first: React will attempt to re-render with the old value. then: try another re-render in background with the new value.
#### How to use
  - showing stale content while fresh content is loading
  - Indicating that the content is stale
  -  Deferring re-rendering for a part of the UI
#### QA
  - [the difference between deferring a value, debounce, throttle](#the-difference-between-deferring-a-value-and-debounce-or-throttling)


## Resource Hooks
### use()
  #### read the value of a resource. like a promise or context

## Other Hooks
### useDebugValue()
  #### customize the label react devTools displays for your customize hook

### useId()
#### let a component associate with a unique ID with itself. Typically used with accessibility APIS.
  - feature:
    * **it should not be used to generate keys in a list.**
#### ```const id = useId()```  
#### How to use
  - generate unique ids for accessibility attributes.
  - generating IDs for several related elements.
  - Specifying a shared prefix for all generated IDs.

### useSyncExternalStore()
#### let a component subscribe to an external store.
#### ```const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapShot?)```
  - **subscribe function:** subscribe to the store and return a function that unsubscribes.The function takes a single **callback** arguments. When the store changes, it should invoke the provided callback.
  - **getSnapshot:** read a snapshot of the data from the store. 
  - **returns:** the snap of the data in the store.

## QA
### the difference between useRef and useState
  - **changing a ref does not trigger re-render. It means refs are perfect for store information that doesn't affect the visual output of your component.**
  - ref can be used to store information and read it later.
  - ref store information between re-renders. the variable decleared with useState can be reseted on every render.
  - the information is local to each copy of your component(unlike the variables outside, which are shared)
#### the difference between useCallback and useMemo
  - useCallback: cache the reference of fn,avoid re-creating fn,
  - useMemo: cache the result of calculatedValue, avoid re-calculated.
#### the difference between deferring a value and (debounce or throttling)
  - debounce means you should wait for the user to stop typing(eg for a second) before updating the list
  - throttle means you should update the list every once a while(eg: at most once a second)
  - deferring a value:
    * it doesn't need a fixed delay. If the response or the user's device is fast, the deffered re-render will happen immediately and wouldn't be noticeable. If the response or the user's device is slow, it will happen slowly.
    * unlike debounce and throttle, deferring a value is interruptible by default.
  - **if the work you are optimizing doesn't happen during rendering, debounce and throttle are still useful. For example,they can let fire fewer network requests. You can also use these techniques together.**  
#### the difference between cache, memo, useMemo
  - **useMemo:** useMemo is used to cache expensive calculation in client component across renders. **useMemo's cache is only local to the component. it can't be shared between component instance.**
  - **cache:** cache is used in server components to memoize work that can be shared across components. cache is also recommend for memoizing data fetches, unlike ```useMemo``` which should only be used for computations.**cache should only be used in Server Components and the cache will be invalidated across server requests.**
  - **memo:** memo is used to prevent a component re-rendering if its props are unchanged. **compared to ```useMemo```, ```memo``` memoizes the component render based on props vs specific computations. Similar to ```useMemo```, the memoized component only caches the last render with the last props valus. Once the props change, the cache invalidates and the components re-renders.**