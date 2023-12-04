# react-组件
## fragment
### ```<></>``` or ```<Fragment key={key}></Fragment>```
  - let you group elements without wrap node.
  - group elements in fragment have no effect on the result of DOM.
#### eg:
  ```JSX
    <>
      <div></div>
      <div></div>
    </>
  ```
## Profiler
### syntax
  ```jsx
  <Profiler id="App" onRender={onRender}>
    <App/>
  </Profiler>
  ``` 

## strictMode
### ```<StrictMode></StrictMode>``` 
#### let you find the common bug in your component early during development.

## Suspense
### let you display a fallback until its children have finish loading.
### syntax
  ```jsx
    <Suspense fallback={<Loading />}>
      <SomeComponent />
    </Suspense>
  ```