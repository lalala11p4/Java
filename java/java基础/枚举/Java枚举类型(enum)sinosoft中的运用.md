## Java枚举类型(enum)sinosoft中的运用

直接上代码：

```java
public enum DBenum {

	/**
	 * 多数据源配置
	 */
	common("公共模块"),prip("保单登记"), aml("反洗钱"), circ("保监会报表"), defaults("默认"), crs("CRS税务上报"), core("核心Lis"), coreBackup("核心备份");
	private String desc;

	private DBenum(String desc) {
		this.desc = desc;
	}

	public String getDesc() {
		return desc;
	}

	/*
	下面是测试代码
	 */
	public static void main(String[] args) {
		System.out.println(DBenum.defaults.desc);
		System.out.println("***********************");
		for(DBenum gender : DBenum.values()){
			System.out.println(gender.desc);
		}
	}
}

```

## Enum 类型的特点

- 在某些情况下，一个类的对象时有限且固定的，如季节类，它只有春夏秋冬4个对象这种实例有限且固定的类，在 Java 中被称为枚举类；
- 在 Java 中使用 enum 关键字来定义枚举类，其地位与 class、interface 相同；
- 枚举类是一种特殊的类，它和普通的类一样，有自己的成员变量、成员方法、构造器 (只能使用 private 访问修饰符，所以无法从外部调用构造器，构造器只在构造枚举值时被调用)；
- 一个 Java 源文件中最多只能有一个 public 类型的枚举类，且该 Java 源文件的名字也必须和该枚举类的类名相同，这点和类是相同的；
- 使用 enum 定义的枚举类默认继承了 java.lang.Enum 类，并实现了 java.lang.Seriablizable 和 java.lang.Comparable 两个接口;
- 所有的枚举值都是 public static final 的，且非抽象的枚举类不能再派生子类；
- 枚举类的所有实例(枚举值)必须在枚举类的第一行显式地列出，否则这个枚举类将永远不能产生实例。列出这些实例(枚举值)时，系统会自动添加 public static final 修饰，无需程序员显式添加。



项目运用的解释：

- 枚举的特点1就说明有限固定长度的类，对于民生的大系统而言，系统的个数是有限的，所以可以使用枚举类来执行

- 在调用时，因为是配置了多数据源，所以就可以使用这里来进行识别：

  ```java
  conn = DBConnectionPool.getConnection(DBenum tDBenum);
  ```

  假如传进来的参数是DBenum.aml，它对应的值也就是aml，在构造多数据源时，DBConnectionMuti中有一个map保存着所有的数据源：

  ```java
  Map<DBenum, DataSource> dbMap = new HashMap<DBenum, DataSource>();
  ```

  这样就可以从这个map中直接拿到这个DataSource了，然后调用DataSource.getConnection()就可以获取链接。