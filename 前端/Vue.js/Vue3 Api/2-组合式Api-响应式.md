# 响应式

## 核心

### `ref()`

接受一个内部值，返回一个响应式的、可更改的 ref 对象，此对象只有一个指向其内部值的属性 `.value`。

- **类型**

  ts

  ```
  function ref<T>(value: T): Ref<UnwrapRef<T>>
  
  interface Ref<T> {
    value: T
  }
  ```

- **详细信息**

  ref 对象是可更改的，也就是说你可以为 `.value` 赋予新的值。它也是响应式的，即所有对 `.value` 的操作都将被追踪，并且写操作会触发与之相关的副作用。