## 选择日期时间

```java
JSpinner year = new JSpinner(new SpinnerDateModel());
year.setValue(new Date());
//设置时间格式
JSpinner.DateEditor editor = new JSpinner.DateEditor(year, "yyyy-MM-dd HH:mm:ss");
year.setEditor(editor);
year.setBounds(34, 67, 219, 22);
contentPane.add(year);
```

## DatePicker

```java
package gui;
   
import java.awt.Color;
import java.awt.Dimension;
import java.awt.Font;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Date;
import java.util.Locale;
   
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JOptionPane;
  
import com.eltima.components.ui.DatePicker;
   
public class TestGUI {
    public static void main(String[] args) {
   
        JFrame f = new JFrame("LoL");
        f.setSize(400, 300);
        f.setLocation(200, 200);
        f.setLayout(null);
   
        final DatePicker datepick;
        datepick = getDatePicker();
   
        f.add(datepick);
   
        JButton b = new JButton("获取时间");
        b.setBounds(137, 183, 100, 30);
        f.add(b);
   
        b.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                JOptionPane.showMessageDialog(f, "获取控件中的日期：" + datepick.getValue());
                System.out.println(datepick.getValue());
            }
        });
   
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
   
        f.setVisible(true);
    }
 
    private static DatePicker getDatePicker() {
        final DatePicker datepick;
        // 格式
        String DefaultFormat = "yyyy-MM-dd HH:mm:ss";
        // 当前时间
        Date date = new Date();
        // 字体
        Font font = new Font("Times New Roman", Font.BOLD, 14);
   
        Dimension dimension = new Dimension(177, 24);
   
        int[] hilightDays = { 1, 3, 5, 7 };
   
        int[] disabledDays = { 4, 6, 5, 9 };
   
        datepick = new DatePicker(date, DefaultFormat, font, dimension);
   
        datepick.setLocation(137, 83);
        datepick.setBounds(137, 83, 177, 24);
        // 设置一个月份中需要高亮显示的日子
        datepick.setHightlightdays(hilightDays, Color.red);
        // 设置一个月份中不需要的日子，呈灰色显示
        datepick.setDisableddays(disabledDays);
        // 设置国家
        datepick.setLocale(Locale.CHINA);
        // 设置时钟面板可见
        datepick.setTimePanleVisible(true);
        return datepick;
    }
}
```

## JXDatePicker

```java
package gui;
 
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Date;
 
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JOptionPane;
 
import org.jdesktop.swingx.JXDatePicker;
 
public class TestGUI {
    public static void main(String[] args) {
 
        JFrame f = new JFrame("LoL");
        f.setSize(400, 300);
        f.setLocation(200, 200);
        f.setLayout(null);
 
        Date date = new Date();
 
        final JXDatePicker datepick = new JXDatePicker();
 
        // 设置 date日期
        datepick.setDate(date);
 
        datepick.setBounds(137, 83, 177, 24);
 
        f.add(datepick);
 
        JButton b = new JButton("获取时间");
        b.setBounds(137, 183, 100, 30);
        f.add(b);
 
        b.addActionListener(new ActionListener() {
 
            @Override
            public void actionPerformed(ActionEvent e) {
                // 获取 日期
                Date d = datepick.getDate();
                JOptionPane.showMessageDialog(f, "获取控件中的日期 :" + d);
 
            }
        });
 
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
 
        f.setVisible(true);
    }
}
```

