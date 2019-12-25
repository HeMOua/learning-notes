# Stream相关函数

## `map`与`peek`

在 Java Stream 中，`peek` 和 `map` 是两个常用的中间操作方法，它们具有一些区别和不同的用途。

1. `peek` 方法：
   - `peek` 方法用于对流中的每个元素执行某个操作，而不会改变流中的元素内容。
   - 它接受一个 `Consumer` 参数，可以执行任意操作，例如打印元素、记录日志等。
   - `peek` 方法返回的仍然是原始的流，不会改变流中的元素。
   - `peek` 方法主要用于调试、观察流中的元素，而不对元素进行转换。

示例代码：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.stream()
       .peek(System.out::println)
       .forEach(System.out::println);
```

上述代码中，`peek` 方法用于在打印每个元素之前输出到控制台，并不会改变流中的元素。

1. `map` 方法：
   - `map` 方法用于对流中的每个元素执行某种转换操作，并返回转换后的结果。
   - 它接受一个 `Function` 参数，将流中的每个元素映射为另一个元素。
   - `map` 方法返回的是一个新的流，其中包含了转换后的元素。
   - `map` 方法主要用于将流中的元素进行转换，生成一个新的流。

示例代码：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> squaredNumbers = numbers.stream()
                                      .map(n -> n * n)
                                      .collect(Collectors.toList());
```

上述代码中，`map` 方法将流中的每个元素进行平方操作，并生成一个新的流，最终收集转换后的结果到 `squaredNumbers` 列表中。

总结：

- `peek` 方法用于观察流中的元素，不进行转换操作，返回原始流。
- `map` 方法用于对流中的元素进行转换，生成一个新的流。
- `peek` 通常用于调试和观察流中的元素，`map` 用于转换流中的元素。