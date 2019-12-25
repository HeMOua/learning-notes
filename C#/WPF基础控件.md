# WPF基础控件

## `TextBlock`

### 属性

+ `TextTrimming`属性：
  + 值设为 `CharacterEllipsis`：让 TextBlock 在没有足够空间的时候显示 `...` 在没有足够空间显示这麽多文字时，这是一种常见的方法。这也很适合用在当空间不足而且你不想使用多行的时候。
  + 值设为 `WordEllipsis`：当空间不足的时候，以最后一个单字为单位显示，就不会有单字只显
    示一部分的问题发生。
+ `TextWrapping`属性：
  + 值设为`Wrap`：让 TextBlock 在没有足够空间显示时自动换行
+ `Foreground`属性：前景色
  + #xxx
  + Blue、Red、...
+ `Padding`属性
+ `Margin`属性
+ `Backgroun`属性

```xaml
<StackPanel>
        <TextBlock Text="wokao"/>
        <TextBlock Text="this is block"/>
        <TextBlock>this is block<LineBreak/>我要换行</TextBlock>
</StackPanel>
```

### 元素块

+ `LineBreak` 标签：在指定的地方手动换行

+ `Bold`标签：加粗

+ `Italic`标签：斜体

+ `Underline`标签：下划线

+ `Hyperlink`标签：超链接

  + `NavigateUri`属性：跳转链接
  +  `RequestNavigate`属性：指定事件处理方法`Hyperlink_RequestNavigate"`

  ```c#
  private void Hyperlink_RequestNavigate(object sender, RequestNavigateEventArgs e)
  {
      Process.Start(e.Uri.AbsoluteUri);
      // Process.Start(new ProcessStartInfo(e.Uri.AbsoluteUri) { UseShellExecute = true});
  }
  ```

+ `Span`标签：块

以上内容须在 `TextBlock` 标签内

## `Label`

```xaml
<StackPanel Margin="20">
    <Label Content="_Name:" Target="{Binding ElementName=txtName}"/>
    <TextBox x:Name="txtName"/>
    <Label Content="_Email:" Target="{Binding ElementName=txtEmail}"/>
    <TextBox x:Name="txtEmail"/>
</StackPanel>
```



自定义 label 图标

```xaml
<StackPanel Margin="20">

    <Label Target="{Binding ElementName=txtName}">
        <StackPanel Orientation="Horizontal">
            <Image Source="http://cdn1.iconfinder.com/data/icons/fatcow/16/bullet_green.png"/>
            <AccessText Text="_Name:"/>
        </StackPanel>
    </Label>
    <TextBox x:Name="txtName"/>
    <Label Target="{Binding ElementName=txtEmail}">
        <StackPanel Orientation="Horizontal">
            <Image Source="http://cdn1.iconfinder.com/data/icons/fatcow/16/bullet_green.png"/>
            <AccessText Text="_Email:"/>
        </StackPanel>
    </Label>
    <TextBox x:Name="txtEmail"/>
</StackPanel>
```

