# ESLint

> ESLint 是一个可配置的 JavaScript linter。它可以帮助你发现并修复 JavaScript 代码中的问题。问题可以是任何事情，从潜在的运行时错误到不遵循最佳实践，再到样式问题。

## 核心概念

### 规则

规则是 ESLint 的核心构建块。规则验证你的代码是否满足特定期望，以及如果不满足该期望该怎么办。规则还可以包含特定于该规则的其他配置选项。

ESLint 包含数百条可供使用的内置规则。还可以创建自定义规则或使用其他人通过 `plugins` 创建的规则。

#### 规则修复

+ 规则可以选择为他们发现的违规行为提供修复。修复安全地纠正违规行为，而无需更改应用逻辑。

+ 修复程序可以通过 `--fix 命令行选项` 或 `编辑器扩展` 自动应用。
+ 可能提供修复的规则在 规则 中用 🔧 标记。

#### 规则建议

+ 除了提供修复之外或代替提供修复之外，规则还可以选择提供建议。建议与修复有两个不同之处：
  + 建议可能会更改应用逻辑，因此无法自动应用。
  + 建议无法通过 ESLint CLI 进行应用，只能通过编辑器集成使用。

+ 可能提供建议的规则在 规则 中用💡标记。

### 配置规则

1、配置文件

ESLint 配置文件是在项目中放置 ESLint 配置的位置。可以包含内置规则、希望它们如何执行、具有自定义规则的插件、可共享配置、希望规则应用到哪些文件等等。

2、共享的配置

共享的配置是指一些可以被其他项目引用并复用的 ESLint 配置文件。这些配置通常包含了一组规则、环境设置、插件等，***目的是让多个项目共享相同的配置***，减少每个项目中重复编写 ESLint 配置的工作。

3、插件

ESLint 插件是一个 npm 模块，可以包含一组 ESLint 规则、配置、处理器和环境。插件通常包含自定义规则。

4、共享的配置和插件的区别

+ **共享的配置** 主要用于统一多个项目的 ESLint 配置，方便维护一致性。

+ **插件** 用于扩展 ESLint 的规则功能，可以为特定框架或库提供支持，或者定义自定义的代码质量规则。

### 解析器

ESLint 解析器将代码转换为 ESLint 可以计算的抽象语法树。默认情况下，ESLint 使用内置的 Espree 解析器，它与标准 JavaScript 运行时和版本兼容。

自定义解析器让 ESLint 解析非标准 JavaScript 语法。通常，自定义解析器作为可共享配置或插件的一部分包含在内，因此你不必直接使用它们。

例如，`@typescript-eslint/parser` 是 typescript-eslint 项目中包含的自定义解析器，可让 ESLint 解析 TypeScript 代码。

### 自定义处理器

ESLint 处理器从其他类型的文件中提取 JavaScript 代码，然后让 ESLint 检测 JavaScript 代码。或者，你可以在使用 ESLint 解析 JavaScript 代码之前使用处理器来操作 JavaScript 代码。

例如，`@eslint/markdown` 包含一个自定义处理器，可让你在 Markdown 代码块内检查 JavaScript 代码。

### 格式化

ESLint 格式化程序控制 CLI 中 linting 结果的外观。

## 词汇表

### 严重性

如果有的话，规则配置为运行什么级别的报告。

ESLint 支持三个严重级别：

+ `"off" (0)`：不要运行该规则。

+ `"warn" (1)`：运行规则，但不要因其违规行为而以非零状态代码退出（不包括 --max-warnings）

+ `"error" (2)`：运行规则，如果产生任何违规，则以非零状态代码退出

### 内联配置

内联配置使用与 配置文件 类似的语法来按名称、新严重性以及可选的规则新选项指定任意数量的规则。

例如，以下内联配置注释同时禁用 eqeqeq 规则并将 curly 规则设置为 "error"：

```js
/* eslint eqeqeq: "off", curly: "error" */
```

