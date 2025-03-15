### **VSCode 通用配置**

这些设置影响 VSCode 编辑器的基本行为和界面。

```json
  "files.autoSaveDelay": 500,  // 自动保存的延迟时间，单位是毫秒
  "files.autoSave": "onFocusChange",  // 自动保存设置，表示在焦点改变时自动保存
  "files.defaultLanguage": "html",  // 设置默认语言为 HTML
  "editor.fontSize": 18,  // 设置编辑器字体大小为 18
  "editor.mouseWheelZoom": true,  // 启用鼠标滚轮缩放功能
  "editor.tabSize": 4,  // 设置编辑器的制表符大小为 4
  "editor.suggestSelection": "first",  // 设置自动完成时选中第一个建议项
  "editor.formatOnSave": false,  // 禁用保存时自动格式化
  "editor.inlineSuggest.enabled": true,  // 启用内联建议功能
  "workbench.startupEditor": "none",  // 启动时不显示任何编辑器
  "workbench.colorTheme": "Default Dark+",  // 设置默认的暗色主题
  "workbench.iconTheme": "vscode-icons",  // 设置 VSCode 图标主题为 vscode-icons
  "workbench.editor.enablePreview": false,  // 禁用编辑器预览模式
  "workbench.editorAssociations": {  // 配置文件类型与编辑器关联
    "*.copilotmd": "vscode.markdown.preview.editor",  // 以 copilotmd 为后缀的文件使用 Markdown 预览编辑器
    "*.md": "vscode.markdown.preview.editor",  // 以 md 为后缀的文件使用 Markdown 预览编辑器
    "{git,gitlens,git-graph}:/**/*.{md,csv,svg}": "default"  // git 相关文件类型默认使用默认编辑器
  },
  "window.openFoldersInNewWindow": "on",  // 每次打开文件夹时都在新窗口中打开
  "search.quickOpen.includeHistory": false,  // 禁用快速打开历史记录
  "explorer.confirmDelete": false,  // 禁用文件删除确认提示
  "settingsSync.ignoredExtensions": ["golang.go"],  // 忽略 golang 扩展的设置同步
  "redhat.telemetry.enabled": true,  // 启用 RedHat 插件的遥测数据收集

```

