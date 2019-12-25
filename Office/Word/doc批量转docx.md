# doc批量转docx

```vb
Dim oWord As Object
Dim oDic As Object
Const wdExportFormatPDF = 17
Sub doc批量转docx()
   Set oWord = VBA.CreateObject("word.application")
    Dim sPath As String
    '获取文件或者文件夹的路径
    sPath = GetPath()
    If Len(sPath) Then
        EnuAllFiles (sPath)
        MsgBox "处理完成!!!"
    End If
    oWord.Quit
    Set oWord = Nothing
End Sub
Function GetPath() As String
    '声明一个FileDialog对象变量
    Dim oFD As FileDialog
'    '创建一个选择文件对话框
'    Set oFD = Application.FileDialog(msoFileDialogFilePicker)
    '创建一个选择文件夹对话框
    Set oFD = Application.FileDialog(msoFileDialogFolderPicker)
    '声明一个变量用来存储选择的文件名
    Dim vrtSelectedItem As Variant
    With oFD
        '允许选择多个文件
        .AllowMultiSelect = True
        '使用Show方法显示对话框，如果单击了确定按钮则返回-1。
        If .Show = -1 Then
            '遍历所有选择的文件
            For Each vrtSelectedItem In .SelectedItems
                '获取所有选择的文件的完整路径,用于各种操作
                GetPath = vrtSelectedItem
            Next
            '如果单击了取消按钮则返回0
        Else
        End If
    End With
    '释放对象变量
    Set oFD = Nothing
End Function
Function GetFileName(ByVal sName As String)
    '获取纯文件名的自定义函数
    '
    Dim sTemp As String
    sTemp = sName
    '判断后缀名分隔符.的位置
    iPos = Len(sTemp) - VBA.InStr(1, VBA.StrReverse(sTemp), ".")
    If iPos <> 0 Then
        sTemp = Mid(sTemp, 1, iPos)
    End If
    '判断路径分隔符\的位置
    iPos = VBA.InStr(1, sTemp, "\")
    If iPos <> 0 Then
        '反转后好取字符
        iPos = VBA.InStr(1, VBA.StrReverse(sTemp), "\")
        sTemp = Mid(VBA.StrReverse(sTemp), 1, iPos - 1)
        sTemp = VBA.StrReverse(sTemp)
    End If
    GetFileName = sTemp
End Function
 
Sub EnuAllFiles(ByVal sPath As String, Optional bEnuSub As Boolean = False)
    '定义文件系统对象
    Dim oFso As Object
    Set oFso = CreateObject("Scripting.FileSystemObject")
    '定义文件夹对象
    Dim oFolder As Object
    Set oFolder = oFso.GetFolder(sPath)
    '定义文件对象
    Dim oFile As Object
    '如果指定的文件夹含有文件
    If oFolder.Files.Count Then
        For Each oFile In oFolder.Files
            With oFile
                '输出文件所在的盘符
                Dim sDrive As String
                sDrive = .Drive
                '输出文件的类型
                Dim sType As String
                sType = .Type
                '输出含后缀名的文件名称
                Dim sName As String
                sName = .Name
                '输出含文件名的完整路径
                Dim sFilePath As String
                sFilePath = .Path
                '如果文件是Word文件且不是隐藏文件
                If sType Like "*ord*2003*" And Not (sName Like "*~$*") Then
                    Const wdFormatDocumentDefault = 16
                    Const wdFormatXMLDocument = 12
                    Const wdWord2013 = 15
                    Set oDoc = oWord.Documents.Open(sFilePath)
                    With oDoc
                        Dim sFPath As String
                        sFPath = .Path
                        sName = .Name
                        sName = GetFileName(sName)
                        sName = sFPath & "\" & sName & ".docx"
                        .SaveAs2 FileName:=sName, FileFormat:=wdFormatDocumentDefault, CompatibilityMode:=wdWord2013
                        .Close
                    End With
                End If
            End With
        Next
    '如果指定的文件夹不含有文件
    Else
    End If
    '如果要遍历子文件夹
    If bEnuSub = True Then
        '定义子文件夹集合对象
        Dim oSubFolders As Object
        Set oSubFolders = oFolder.SubFolders
        If oSubFolders.Count > 0 Then
            For Each oTempFolder In oSubFolders
                sTempPath = oTempFolder.Path
                Call EnuAllFiles(sTempPath, True)
            Next
        End If
        Set oSubFolders = Nothing
    End If
    Set oFile = Nothing
    Set oFolder = Nothing
    Set oFso = Nothing
End Sub
```

