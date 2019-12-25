# 设置Tab宽度

来源：http://www.voidcn.com/article/p-bfthkoua-bvc.html

当我在TextArea中使用制表(Tab键)时,制表的宽度很宽.我想控制宽度,即使用4个空格.在文档中,我找不到一种方法来执行此操作.

我尝试了这个代码(其中taInput是一个TextArea),但它不能正常工作：

```java
taInput.setOnKeyPressed(new EventHandler<KeyEvent>() {
    @Override
    public void handle(KeyEvent e) {
        if (e.getCode() == KeyCode.TAB) {
            // TAB SPACES
            StringBuilder sb = new StringBuilder(config.getTabSpacesCount());
            for (int i=0; i<config.getTabSpacesCount(); i++) {
                sb.append(' ');
            }
            taInput.insertText(taInput.getCaretPosition(), sb.toString());
            e.consume();
        }
    }
});
```

最后我找到了一种方法来做到这一点.

似乎setOnKeyPressed()方法不适合此任务,因为事件是在执行keyPress操作后处理的.

addEventFilter()在执行操作之前处理事件,因此您可以操作事件.

我的新代码：

```java
taInput.addEventFilter(KeyEvent.KEY_PRESSED, new EventHandler<KeyEvent>() {
    @Override
    public void handle(KeyEvent e) {
        if (e.getCode() == KeyCode.TAB) {
            String s = StringUtils.repeat(' ', config.getTabSpacesCount());
            taInput.insertText(taInput.getCaretPosition(), s);
            e.consume();
        }
    }
});
```



封装后

```java
public class TabEventHandler implements EventHandler<KeyEvent> {

    private int spaceNum = 4;
    private final CodeArea codeArea;

    public TabEventHandler(CodeArea codeArea) {
        this.codeArea = codeArea;
    }

    public TabEventHandler(CodeArea codeArea, int spaceNum) {
        this.spaceNum = spaceNum;
        this.codeArea = codeArea;
    }

    @Override
    public void handle(KeyEvent e) {
        if (e.getCode() == KeyCode.TAB) {
            StringBuilder space = new StringBuilder();
            for (int i = 0; i < spaceNum; i++) space.append(" ");
            int start = codeArea.getCaretPosition();
            codeArea.insertText(start, space.toString());
            e.consume();
        }
    }
}
```

