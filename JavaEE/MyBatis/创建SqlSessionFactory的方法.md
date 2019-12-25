# 创建SqlSessionFactory的方法

## 纯代码

```java
package ssm;
 
import java.sql.SQLException;
 
import javax.sql.DataSource;
 
import org.apache.ibatis.datasource.pooled.PooledDataSource;
import org.apache.ibatis.mapping.Environment;
import org.apache.ibatis.session.Configuration;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.apache.ibatis.transaction.TransactionFactory;
import org.apache.ibatis.transaction.jdbc.JdbcTransactionFactory;
 
import com.sm.model.User;
 
public class GetSqlSessionFactoryFromProgram {
 
	public static void main(String[] args) throws SQLException {
		String driver = "oracle.jdbc.OracleDriver";
		String url = "jdbc:oracle:thin:@localhost:1521:orcl";
		String username="scott";
		String password="tiger";
		//创建使用缓存池的数据源
		/* 
		 * <dataSource type="POOLED">
				<property name="driver" value="${jdbc.driverClassName}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		 */
		DataSource dataSource =new PooledDataSource(driver,url,username,password);
		
		//创建事务
		/*
		 * <transactionManager type="JDBC" />
		 */
		TransactionFactory transactionFactory =  new JdbcTransactionFactory();
		
		Environment environment = new Environment("development", transactionFactory, dataSource);
		
		Configuration configuration = new Configuration(environment);
		//加入资源
		/*
		 * <mapper resource="ssm/BlogMapper.xml"/>
		 */
		configuration.addMapper(UserMapper.class);
		SqlSessionFactory sqlSessionFactory = new  SqlSessionFactoryBuilder().build(configuration);
		System.out.println(sqlSessionFactory);
		
		SqlSession session =  sqlSessionFactory.openSession();
		User user = new User();
		user.setId(6);
 		session.insert("add", user);
 		session.commit();
		session.close();
	}
}
```

