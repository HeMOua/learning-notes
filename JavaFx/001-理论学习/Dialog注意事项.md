# Dialog 注意事项



```java
public class MyDialog extends Dialog<ButtonType> {

    public MyDialog(Window root, Salary obj, String title) {
        this.obj = obj;
        initOwner(root);
        setTitle(title);
        setHeaderText(null);
        initView();
    }

    private void initView() {
        DialogPane dialogPane = getDialogPane();
        dialogPane.setPrefSize(320, 200);
        
        // 面板z内容
        dialogPane.setContent(mainPanel());

        // 按钮
        ObservableList<ButtonType> buttonTypes = dialogPane.getButtonTypes();
        buttonTypes.addAll(ButtonType.OK, ButtonType.CANCEL);
        Button btnOk = (Button) dialogPane.lookupButton(ButtonType.OK);
        btnOk.addEventFilter(ActionEvent.ACTION, e -> {
            try {
				...
            } catch (Exception ex) {
	
                e.consume();
            }
        });
    }
}
```

我们通过 `dialogPane.getButtonTypes()` 为 Dialog 添加按钮后，如果我们点击按钮，Dialog 就会自动销毁，有时需要校验数据，不能让窗口立即销毁。

可以通过 `btnOk.addEventFilter()` 方法，然后再调用 `e.consume()` 这样事件就会被吃掉，窗口就不会自动销毁了

