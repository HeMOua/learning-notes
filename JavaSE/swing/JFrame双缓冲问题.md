# JFrame双缓冲问题

## 问题

不是说swing有双缓冲么，为什么这样不行啊

```java
public class MainFrame extends JFrame implements KeyListener{
     
    int x = 0;
    int y = 0;
     
    public void paint(Graphics g){
        g.setColor(Color.red);
        g.drawRect(x, y, 40, 30);
    }
     
    public void update(Graphics g) {
         
        paint(g);
 
    }
     
    public void LunchFrame(){
        this.setResizable(false);
        this.setSize(512, 600);
        this.setBackground(Color.black);
        this.setTitle("飞机大战暴爽版！       xsx");
        this.setDefaultCloseOperation(DISPOSE_ON_CLOSE);
        this.setVisible(true);
        this.addKeyListener(this);
         
        new Thread(new PaintThread()).start();
    }
     
    public static void main(String[] args){
        MainFrame mframe = new MainFrame();
        mframe.LunchFrame();
    }
     
    private class PaintThread implements Runnable {
 
        public void run() {
            while(true) {
                repaint();
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
     
    @Override
    public void keyTyped(KeyEvent e) {
        // TODO Auto-generated method stub
         
    }
 
    @Override
    public void keyPressed(KeyEvent e) {
        // TODO Auto-generated method stub
         int keyCode = e.getKeyCode();
            //后面会添加具体按键匹配
            switch (keyCode) {
            case KeyEvent.VK_SPACE:
                 
                break;
            case KeyEvent.VK_A:
                System.out.println("xxsx");
                x -= 1;
                break;
            case KeyEvent.VK_D:
                x += 1;
                break;
            case KeyEvent.VK_W:
                y -= 1;
                break;
            case  KeyEvent.VK_S:
                y += 1;
                break;
            }
    }
 
    @Override
    public void keyReleased(KeyEvent e) {
        // TODO Auto-generated method stub
         
    }
}
```

## 解答

不要直接重写JFrame的paint方法。你应该使用JPanel，然后重写它的paintComponent(Graphics g)方法。

paintComponent第一行你需要调用父类的paintComponent方法，它会进行擦除，否则会有残影，你可以去掉试试。

然后你的update方法目前看起来是可以不用写的。无用代码。

下面给出修改后代码，直接粘贴可以运行

```java
import java.awt.Color;
import java.awt.Graphics;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
 
import javax.swing.JFrame;
import javax.swing.JPanel;
 
public class MainPanel extends JPanel implements KeyListener {
 
    private static final long    serialVersionUID    = 1147736728548400926L;
     
    int    x    = 0;
    int    y    = 0;
 
    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.red);
        g.drawRect(x, y, 40, 30);
    }
 
    public static void main(String[] args) {
        JFrame f = new JFrame();
        MainPanel mPanel = new MainPanel();
        f.add(mPanel);
 
        f.setResizable(false);
        f.setSize(512, 600);
        f.setBackground(Color.black);
        f.setTitle("飞机大战暴爽版！       xsx");
        f.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
        f.setVisible(true);
 
        mPanel.addKeyListener(mPanel);
        mPanel.start();
        mPanel.requestFocus();
    }
 
    private void start() {
        new Thread(new PaintThread()).start();
    }
 
    private class PaintThread implements Runnable {
        public void run() {
            while (true) {
                repaint();
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
 
    @Override
    public void keyTyped(KeyEvent e) {
    }
 
    @Override
    public void keyPressed(KeyEvent e) {
        int keyCode = e.getKeyCode();
        switch (keyCode) {
        case KeyEvent.VK_SPACE:
            break;
        case KeyEvent.VK_A:
            System.out.println("xxsx");
            x -= 1;
            break;
        case KeyEvent.VK_D:
            x += 1;
            break;
        case KeyEvent.VK_W:
            y -= 1;
            break;
        case KeyEvent.VK_S:
            y += 1;
            break;
        }
    }
 
    @Override
    public void keyReleased(KeyEvent e) {
    }
}
```

## 拓展知识

1 - **paint**() 中调用 paintComponent(), paintBorder(), paintChildren()

2 - 最重要的区别是“双缓冲”。**Swing** 组件的 **paint**() 中实现了双缓冲，所以不要随便去覆写，会破坏双缓冲的，————建议的方式是覆写 paintComponent()，很多人做的小程序会”闪烁“，就是因为他们覆写了 **paint**() 方法，破坏了**Swing**本身的双缓冲。**Swing** 不建议用户自己实现双缓冲。

3 - 覆写 **paint**()，如果新方法没有去调用 paintChildren()，还会造成子控件不显示，鼠标移上去才显示，这个也是很多新手问的问题： “为什么我的按钮只有鼠标移上去才显示？”

4 - 只有极少数的情况可能需要覆写 **paint**() 方法，通常是为了实现特殊的绘图效果，或者特殊的优化，比如 JViewport 覆写了 **paint**() 方法，使用“延迟重绘”的方式来合并当滚动条移动时一些特别频繁的重绘请求，等等。