## 如何配置单一数据源和多数据源

### 一、单一数据源

```java
//配置文件，启动的时候即可注入spring
@Configuration
public class SystemConfig {
	private static final Logger logger = LoggerFactory.getLogger(SystemConfig.class);
    
	@Bean(name = "default")
	public DataSource getDataSource() throws PropertyVetoException, NamingException, IOException {
		//获取流，并且写入Properties
		Properties properties = new Properties();
		InputStream in = SystemConfig.class.getResourceAsStream("/jdbc.properties");
		properties.load(in);

		logger.info("加载jdbc.properties参数");
		// 数据源初始化放入Bean
		Map<String, String> commonMap = new HashMap<String, String>();
		for (String keyName : properties.stringPropertyNames()) {
			String value = properties.getProperty(keyName);

			commonMap.put(keyName, value);
		}
		logger.info("jdbc.properties参数加载完毕");

		return DBOper.registeDataSource(commonMap);
	}

}
```

```java
public static DataSource registeDataSource(Map<String, String> propMap)
			throws PropertyVetoException, NamingException {

		/*
		jdbc的书写样式，传入的map和这个一样
        DBType:oracle
        IP:*****
        Port:**
        DBName:sinosoft
        DBUser:****
        PassWord:****
        MaxActive:100
        MaxIdle:24000000
        JdbcUrl:
        Driver:
		*/
    
		System.out.println("=================开始注入数据源");
		String dbType = propMap.get("DBType");

    	//https://www.cnblogs.com/superroshan/articles/4692490.html对于			
    	//new  InitialContext()的理解
		if ("DataSource".equalsIgnoreCase(dbType)) {
			Context tContext = new InitialContext();
			return (DataSource) tContext.lookup(propMap.get("DBName"));
		} else {
            //通过c3p0数据池来配置数据池
			ComboPooledDataSource dataSource = new ComboPooledDataSource();
			dataSource.setDriverClass(DBOper.getDriver(dbType, propMap.get("Driver")));
			dataSource.setJdbcUrl(getUrl(propMap));
			dataSource.setUser(propMap.get("DBUser"));
			dataSource.setPassword(propMap.get("PassWord"));

			// 数据库连接的最大空闲时间(毫秒)
			String dbMaxIdle = propMap.get("MaxIdle");
			if (!StringUtils.isEmpty(dbMaxIdle)) {
				dataSource.setMaxIdleTime(Integer.parseInt(dbMaxIdle));
			}

			// 连接池的最大数据库连接数
			String dbMaxActive = propMap.get("MaxActive");
			if (!StringUtils.isEmpty(dbMaxActive)) {
				dataSource.setMaxPoolSize(Integer.parseInt(dbMaxActive));
			}

			// c3p0强制回收超时(秒)
			String unreturnedConnTs = propMap.get("UnreturnedConnectionTimeout");
			if (!StringUtils.isEmpty(unreturnedConnTs)) {
				dataSource.setUnreturnedConnectionTimeout(Integer.parseInt(unreturnedConnTs));
			}

			// c3p0强制回收连接时输出日志(限测试环境)
			String debugUnreturnedConnTs = propMap.get("DebugUnreturnedConnectionStackTraces");
			if (!StringUtils.isEmpty(debugUnreturnedConnTs)) {
				dataSource.setDebugUnreturnedConnectionStackTraces(Boolean.valueOf(debugUnreturnedConnTs));
			}
			return dataSource;
		}
	}
```

配置完成！

```java
一、
conn = DBConnectionPool.getConnection();
二、
public static Connection getConnection() throws SQLException{
		Connection conn;
		try{
            //单一数据源，到第三步
			conn = BeanUtils.getBean("default", DataSource.class).getConnection();
		}catch(Exception e){
            //多数据源
			conn = DBConnectionMuti.getInstanse().getDataSource(DBenum.defaults).getConnection();
		}
		return conn;
	}
三、	
    public static <T> T getBean(String name, Class<T> clazz) {
        ApplicationContext applicationContext;
        return applicationContext.getBean(name, clazz);
    }
```

调用完成！



## 二，多数据源配置

```java
DBConnectionMuti.getInstanse().getDataSource(DBenum.defaults).getConnection();
			1 						2								3
```

- 1是DBConnectionMuti为单例

- 2是DBConnectionMuti类中有成员变量如下map，从map中取DataSource

  ```java
  private Map<DBenum, DataSource> dbMap = new HashMap<DBenum, DataSource>();
  ```

- 3是DataSource.getConnection();



但是在项目中，多数据源在哪里配置的呢？答案就在DBConnectionMuti（）的构造函数中

```java
 private DBConnectionMuti() {

        Properties properties = new Properties();
        InputStream in = SystemConfig.class.getResourceAsStream("/jdbc.properties");
        try {
            properties.load(in);
        } catch (Exception e) {
            logger.error("配置文件加载错误", e);
        }
        // 数据源初始化放入Bean
        Map<String, String> commonMap = new HashMap<String, String>();
        for (String keyName : properties.stringPropertyNames()) {
            String value = properties.getProperty(keyName);
            commonMap.put(keyName, value);
        }
        //到这里和上面一样，把jdbc里面的数据放到commonMap中
        /*
            for (DBenum dbconn : DBenum.values()) {
                        String sign = dbconn.toString();
                        System.out.println(sign);
             }
             结果如下（项目编号）：
            common
            prip
            aml
            defaults
        */

        for (DBenum dbconn : DBenum.values()) {
            String sign = dbconn.toString();
            if (dbconn.equals(DBenum.defaults)) {
                sign = "";
            }
            

            String dbtype = commonMap.get(sign + "DBType");

            if (StringUtils.isEmpty(dbtype)) {
                logger.debug("未配置数据源" + sign);
                continue;
            }

            if ("DataSource".equalsIgnoreCase(dbtype)) {
                try {
                    Context tContext = new InitialContext();
                    DataSource ds = (DataSource) tContext.lookup(commonMap.get(sign + "DBName"));
                    if (ds != null) {
                        dbMap.put(dbconn, ds);
                    }
                } catch (Exception e) {
                    logger.error("数据源未获取" + sign, e);
                }
            } else {
                ComboPooledDataSource dataSource = new ComboPooledDataSource();
                try {
                    dataSource.setDriverClass(DBOper.getDriver(dbtype, commonMap.get(sign + "Driver")));
                    dataSource.setJdbcUrl(getUrl(commonMap.get(sign + "JdbcUrl"), commonMap.get(sign + "DBType"),
                            commonMap.get(sign + "IP"), commonMap.get(sign + "Port"), commonMap.get(sign + "DBName")));
                    dataSource.setUser(commonMap.get(sign + "DBUser"));
                    dataSource.setPassword(commonMap.get(sign + "PassWord"));
                    // 数据库连接的最大空闲时间(毫秒
                    String dbMaxIdle = commonMap.get(sign + "MaxIdle");
                    // 连接池的最大数据库连接数
                    String dbMaxActive = commonMap.get(sign + "MaxActive");
                    if (!StringUtils.isEmpty(dbMaxIdle)) {
                        dataSource.setMaxIdleTime(Integer.parseInt(dbMaxIdle));
                    }
                    if (!StringUtils.isEmpty(dbMaxActive)) {
                        dataSource.setMaxPoolSize(Integer.parseInt(dbMaxActive));
                    }

                    // c3p0强制回收超时(秒)
                    String unreturnedConnTs = commonMap.get(sign + "UnreturnedConnectionTimeout");
                    if (!StringUtils.isEmpty(unreturnedConnTs)) {
                        dataSource.setUnreturnedConnectionTimeout(Integer.parseInt(unreturnedConnTs));
                    }

                    // c3p0强制回收连接时输出日志(限测试环境)
                    String debugUnreturnedConnTs = commonMap.get(sign + "DebugUnreturnedConnectionStackTraces");
                    if (!StringUtils.isEmpty(debugUnreturnedConnTs)) {
                        dataSource.setDebugUnreturnedConnectionStackTraces(Boolean.valueOf(debugUnreturnedConnTs));
                    }
                    dbMap.put(dbconn, dataSource);
                } catch (Exception e) {
                    logger.error("数据源未获取" + sign, e);
                }

            }

        }
    }
```

over!