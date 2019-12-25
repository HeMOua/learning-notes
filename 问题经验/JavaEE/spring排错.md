## Mybatis整合

1）出现NoClassDefFoundError: DaoSupport异常

mybatis和spring整合时出现java.lang.NoClassDefFoundError: org/springframework/dao/support/DaoSupport

解决：由于**缺少spring-tx包**

2）出现NoClassDefFoundError: TransactionAwareDataSourceProxy异常

Caused by: org.springframework.beans.PropertyBatchUpdateException; nested PropertyAccessExceptions (1) are:
PropertyAccessException 1: org.springframework.beans.MethodInvocationException: Property 'dataSource' threw exception; nested exception is java.lang.NoClassDefFoundError: org/springframework/jdbc/datasource/TransactionAwareDataSourceProxy

解决：**缺少spring-jdbc包**

小结：spring-jdbc包依赖spring-tx包，不管使用什么连接池或者什么dao层技术都需要导入spring-jdbc包