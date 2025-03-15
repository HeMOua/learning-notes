# 配置 ESLint

## 配置文件

ESLint 配置文件可以命名为以下任意名称：

+ js 类：`eslint.config.(m|c)js`
+ ts 类：`eslint.config.(m|c)ts`

其中 `m` 表示 ES Module，使用 import、export 导入导出，使用这种配置需要在 package.json 指定 `"type": "module"`。

`c` 则表示 CommonJS，使用 require、module.export 导入导出，默认则是这种情况

## 配置对象

每个配置对象都包含 ESLint 需要在一组文件上执行的所有信息。每个配置对象都由以下属性组成：

- `name` - 配置对象的名称。这在错误消息和配置检查器中使用，以帮助识别正在使用哪个配置对象。
- `files` - 指示配置对象应应用于的文件的通配符模式数组。如果未指定，则配置对象适用于与任何其他配置对象匹配的所有文件。
- `ignores` - 指示配置对象不应应用于的文件的通配符模式数组。如果未指定，则配置对象适用于 `files` 匹配的所有文件。如果在配置对象中使用 `ignores` 而没有任何其他键，则模式将充当 [全局忽略](https://eslint.nodejs.cn/docs/latest/use/configure/configuration-files#globally-ignoring-files-with-ignores)。
- `languageOptions` - 包含与如何为代码检查配置 JavaScript 相关的设置的对象。
  - `ecmaVersion` - 支持的 ECMAScript 版本。可以是任何年份（即 `2022`）或版本（即 `5`）。对于最新支持的版本，设置为 `"latest"`。（默认：`"latest"`）
  - `sourceType` - JavaScript 源代码的类型。可能的值是 `"script"` 用于传统脚本文件，`"module"` 用于 ECMAScript 模块 (ESM) 和 `"commonjs"` 用于 CommonJS 文件。（默认：`"module"` 用于 `.js` 和 `.mjs` 文件；`"commonjs"` 用于 `.cjs` 文件）
  - `globals` - 指定在代码检查期间应添加到全局作用域的其他对象的对象。
  - `parser` - 包含 `parse()` 方法或 `parseForESLint()` 方法的对象。（默认：[`espree`](https://github.com/eslint/js/tree/main/packages/espree)）
  - `parserOptions` - 指定直接传递给解析器上的 `parse()` 或 `parseForESLint()` 方法的其他选项的对象。可用选项取决于解析器。
- `linterOptions` - 包含与代码检查过程相关的设置的对象。
  - `noInlineConfig` - 一个布尔值，指示是否允许内联配置。
  - `reportUnusedDisableDirectives` - 一个严重性字符串，指示是否以及如何应跟踪和报告未使用的禁用和启用指令。对于旧版兼容性，`true` 相当于 `"warn"`，`false` 相当于 `"off"`。（默认值：`"warn"`）。
- `processor` - 包含 `preprocess()` 和 `postprocess()` 方法的对象或指示插件内处理器名称的字符串（即 `"pluginName/processorName"`）。
- `plugins` - 包含插件名称到插件对象的名称-值映射的对象。指定 `files` 时，这些插件仅对匹配的文件可用。
- `rules` - 包含配置规则的对象。当指定 `files` 或 `ignores` 时，这些规则配置只对匹配的文件可用。
- `settings` - 一个包含名称-值对信息的对象，所有规则都应使用这些信息。