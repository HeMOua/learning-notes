# SpringBoot相关

## ImportBeanDefinitionRegistrar

> ImportBeanDefinitionRegistrar动态注册bean

+ ImportBeanDefinitionRegistrar类只能通过其他类@Import的方式来加载，通常是启动类或配置类。
+ 使用@Import，如果括号中的类是ImportBeanDefinitionRegistrar的实现类，则会调用接口方法，将其中要注册的类注册成bean。
+ 实现该接口的类拥有注册bean的能力。
  