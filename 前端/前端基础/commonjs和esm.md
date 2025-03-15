# CommonJS 和 ESM 的区别

## **`cjs`** - CommonJS 模块

- **CommonJS (CJS)** 是一种在 Node.js 中广泛使用的模块系统。它使用 `require` 来导入模块，`module.exports` 或 `exports` 来导出模块。

- **文件扩展名**：`.cjs` 表示该文件使用 CommonJS 模块系统。

- 示例

  ```js
  // math.cjs
  module.exports = {
    add: (a, b) => a + b,
  };
  ```

## **`mjs`** - ES Module 模块

- **ES Module (ESM)** 是 JavaScript 的标准模块系统，使用 `import` 和 `export` 语法。它在浏览器和现代的 Node.js 中都有广泛的支持。

- **文件扩展名**：`.mjs` 表示该文件使用 ES Module 模块系统。

- 示例

  ```js
  // math.mjs
  export function add(a, b) {
    return a + b;
  }
  ```

## 为什么使用 `.cjs` 和 `.mjs`？

在 Node.js 中，不同的模块系统（CommonJS 和 ES Module）在行为上有所不同。例如，`import` 和 `require` 在语法和执行顺序上有差异。因此，Node.js 使用这两种不同的扩展名来区分这两种模块类型，帮助开发者明确文件使用的模块系统。

## **`.js`** - 普通 js 脚本

- `.js`  是普通的 JavaScript 文件扩展名，但在一些环境中，Node.js 会根据项目的 `package.json` 中的配置（如 `"type": "module"`）来决定 `.js` 文件是否为 ESM 模块。
  - 如果 `package.json` 中没有指定 `"type"`，Node.js 会默认将 `.js` 文件视为 CommonJS 模块。
  - 如果 `package.json` 中设置了 `"type": "module"`，则 `.js` 文件被视为 ES Module。