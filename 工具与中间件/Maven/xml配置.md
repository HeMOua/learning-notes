## 层级

```
└──project
	├──properties
	├──dependencies
	└──build
		└──plugins
			└──plugin
		└──resources
			└──resource

```

## properties

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
</properties>
```

## resources

```xml
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.png</include>
        </includes>
    </resource>
    <resource>
        <directory>src/main/resources</directory>
        <includes>
            <include>**/*.png</include>
        </includes>
    </resource>
</resources>
```

