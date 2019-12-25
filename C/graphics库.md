# graphics库

1、initgraph（int width，int height）//初始化一个窗口

2、closegraph（） ；			//关闭窗口

3、cleardevice（）；			//用设定好的背景颜色刷新屏幕  

4、setbkcolor（c）；			//设置背景颜色

5、图形
	圆形 circle（x,y,r）
	矩形 rectangle（X1,Y1,X2,Y2）
	填充的图形 fill**  

6、setlinecolor（c）			//设置线的颜色

7、putpixel（x,y,c）			//画点，颜色c

8、MOUSEMSG m				//定义鼠标变量

​      m=GetMouseMsg(); 	  //获取鼠标信息

mousemsg结构体介绍	

```c
struct MOUSEMSG
{
    UINT uMsg;      // 当前鼠标消息

    bool mkCtrl;    // Ctrl 键是否按下

    bool mkShift;   // Shift 键是否按下

    bool mkLButton; // 鼠标左键是否按下

    bool mkMButton; // 鼠标中键是否按下

    bool mkRButton; // 鼠标右键是否按下

    int x;          // 当前鼠标 x 坐标

    int y;          // 当前鼠标 y 坐标

    int wheel;      // 鼠标滚轮滚动值
};
```

9、IMAGE img//定义图片对象

​    loadimage（&img，“地址及图片名”）			//读取图片到对象img中

​    putimage（0，0，&img）					//在坐标（0,0）位置显示图片

10、getimage（&img，int x,int y,int width,int heigh）	//获取屏幕图像

11、outtextxy（int x,int y,"你好"）				//输出文字“你好”

12、settextstyle(大小，0，类型)				//设置文字大小，类型

13、setfillcolor(c)						//设置覆盖颜色

14，setbkmode(transparent)					//设置文字背景为透明

15、kbhit（）							//检查是否有按键 头文件conio.h