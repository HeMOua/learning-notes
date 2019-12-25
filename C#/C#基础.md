# C#基础

Console.writeLine()  输出换行
Console.write()         输出不换行
Console.ReadLine()  返回string
Console.Read()         返回int

readonly 是运行时常量，只有在运行时才能进行复制

●#region 和 #endregion  功能：折叠代码

●#define 和 #undef      功能：定义和取消符号，通常用于为#if指令传递条件表达式，#define的结果为true，#undef 的结果为false，并且只能在顶部使用

●#if、 #elif、 #else 、 #endif主要用来判断符号是否已经定义。结合使用#if、#else、#elif、#endif指令，可以根据一个或多个符号是否存在来包含或排除代码，这在编译调试版本的代码或针对特定配置进行代码编译时，非常有用。
#if块不能与#region块重叠，但是，可以将#region块嵌套在#if块内，或将#if块嵌套在#region块内

●#warning和#error
#warning"生成警告信息"
#error“生成错误信息”

●#line

●#pragma
为编译器提供特殊的指令，以说明如何编译包含杂注的文件
#pragma warning：启用或禁用某些警告
#pragma warning disable warning-list  禁用
#pragma warning restore warning-list  启用
#pragma checksum：生成源文件的校验和，以帮助调试ASP.NET页
#pragma checksum"filename""{guid}""checksum bytes"

使用预处理指令时不能加分号

显示类型转换
convert.To类型名（） 能显示错误

单目运算符、条件运算符、三目运算符是右结合性
单目>算数>移位>关系>逻辑>三目>赋值

Length：获取数组长度
Rank：获取数组维度