
Your `SQLException` clearly states that:

> You need to specify `Statement.RETURN_GENERATED_KEYS` to the `Statement.executeUpdate()` or `Connection.prepareStatement()`.

This can be achieved as follows (adding an additional value on `Connection.prepareStatement()` method):

```java
String SQL = ""; //whatever my String is
PreparedStatement ps = connection.prepareStatement(SQL, Statement.RETURN_GENERATED_KEYS);
ps.setString(1, "value");
//Other necessary ps.setXXX() methods

//now update
ps.executeUpdate();

ResultSet rs = ps.getGeneratedKeys();
```