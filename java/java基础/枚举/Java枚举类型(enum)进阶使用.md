## Java枚举类型(enum)进阶使用

​		在前面的分析中，我们都是基于简单枚举类型的定义，也就是在定义枚举时只定义了枚举实例类型，并没定义方法或者成员变量，实际上使用关键字enum定义的枚举类，除了不能使用继承(因为编译器会自动为我们继承Enum抽象类而Java只支持单继承，因此枚举类是无法手动实现继承的)，可以把enum类当成常规类，也就是说我们可以向enum类中添加方法和变量，甚至是mian方法，下面就来感受一把。

### 向enum类添加方法与自定义构造函数

​		重新定义一个日期枚举类，带有desc成员变量描述该日期的对于中文描述，同时定义一个getDesc方法，返回中文描述内容，自定义私有构造函数，在声明枚举实例时传入对应的中文描述，代码如下：

```java
public enum Day2 {
    MONDAY("星期一"),
    TUESDAY("星期二"),
    WEDNESDAY("星期三"),
    THURSDAY("星期四"),
    FRIDAY("星期五"),
    SATURDAY("星期六"),
    SUNDAY("星期日");//记住要用分号结束

    private String desc;//中文描述

    /**
     * 私有构造,防止被外部调用
     * @param desc
     */
    private Day2(String desc){
        this.desc=desc;
    }

    /**
     * 定义方法,返回描述,跟常规类的定义没区别
     * @return
     */
    public String getDesc(){
        return desc;
    }

    public static void main(String[] args){
        for (Day2 day:Day2.values()) {
            System.out.println("name:"+day.name()+
                    ",desc:"+day.getDesc());
        }
    }

    /**
     输出结果:
     name:MONDAY,desc:星期一
     name:TUESDAY,desc:星期二
     name:WEDNESDAY,desc:星期三
     name:THURSDAY,desc:星期四
     name:FRIDAY,desc:星期五
     name:SATURDAY,desc:星期六
     name:SUNDAY,desc:星期日
     */
}
```

​		从上述代码可知，在enum类中确实可以像定义常规类一样声明变量或者成员方法。但是我们必须注意到，如果打算在enum类中定义方法，务必在声明完枚举实例后使用分号分开，倘若在枚举实例前定义任何方法，编译器都将会报错，无法编译通过，同时即使自定义了构造函数且enum的定义结束，我们也永远无法手动调用构造函数创建枚举实例，毕竟这事只能由编译器执行。

​	更详细的用法如下：

```java
public class TestEnum {
    /**
     * 普通枚举
     */
    public enum ColorEnum {
        red, green, yellow, blue;
    }
    
    /**
     * 枚举像普通的类一样可以添加属性和方法，可以为它添加静态和非静态的属性或方法
     */
    public enum SeasonEnum {
        //注：枚举写在最前面，否则编译出错
        spring, summer, autumn, winter;

        private final static String position = "test";

        public static SeasonEnum getSeason() {
            if ("test".equals(position))
                return spring;
            else
                return winter;
        }
    }
    
    /**
     * 性别
     * 实现带有构造器的枚举
     */
    public enum Gender{
        //通过括号赋值,而且必须带有一个参构造器和一个属性跟方法，否则编译出错
        //赋值必须都赋值或都不赋值，不能一部分赋值一部分不赋值；如果不赋值则不能写构造器，赋值编译也出错
        MAN("MAN"), WOMEN("WOMEN");
        
        private final String value;

        //构造器默认也只能是private, 从而保证构造函数只能在内部使用
        Gender(String value) {
            this.value = value;
        }
        
        public String getValue() {
            return value;
        }
    }
    
   /**
    *与常规抽象类一样，enum类允许我们为其定义抽象方法，然后使每个枚举实例都实现该方法，以便产生不同的行为	 *方式，注意abstract关键字对于枚举类来说并不是必须的
    * 订单状态
    * 实现带有抽象方法的枚举
    */
    public enum OrderState {
        /** 已取消 */
        CANCEL {public String getName(){return "已取消";}},
        /** 待审核 */
        WAITCONFIRM {public String getName(){return "待审核";}},
        /** 等待付款 */
        WAITPAYMENT {public String getName(){return "等待付款";}},
        /** 正在配货 */
        ADMEASUREPRODUCT {public String getName(){return "正在配货";}},
        /** 等待发货 */
        WAITDELIVER {public String getName(){return "等待发货";}},
        /** 已发货 */
        DELIVERED {public String getName(){return "已发货";}},
        /** 已收货 */
        RECEIVED {public String getName(){return "已收货";}};
        
        public abstract String getName();
    }
    
    //关于枚举与switch是个比较简单的话题，使用switch进行条件判断时，条件参数一般只能是整型，字符型。而枚	   举型确实也被switch所支持，在java 1.7后switch也对字符串进行了支持。这里我们简单看一下switch与枚       举类型的使用：
    public static void main(String[] args) {
        //枚举是一种类型，用于定义变量，以限制变量的赋值；赋值时通过“枚举名.值”取得枚举中的值
        ColorEnum colorEnum = ColorEnum.blue;
        switch (colorEnum) {
        case red:
            System.out.println("color is red");
            break;
        case green:
            System.out.println("color is green");
            break;
        case yellow:
            System.out.println("color is yellow");
            break;
        case blue:
            System.out.println("color is blue");
            break;
        }
        
        //遍历枚举
        System.out.println("遍历ColorEnum枚举中的值");
        for(ColorEnum color : ColorEnum.values()){
            System.out.println(color);
        }
        
        //获取枚举的个数
        System.out.println("ColorEnum枚举中的值有"+ColorEnum.values().length+"个");
        
        //获取枚举的索引位置，默认从0开始
        System.out.println(ColorEnum.red.ordinal());//0
        System.out.println(ColorEnum.green.ordinal());//1
        System.out.println(ColorEnum.yellow.ordinal());//2
        System.out.println(ColorEnum.blue.ordinal());//3
        
        //枚举默认实现了java.lang.Comparable接口
        System.out.println(ColorEnum.red.compareTo(ColorEnum.green));//-1
        
        //--------------------------
        System.out.println("===========");
        System.err.println("季节为" + SeasonEnum.getSeason());
        
        
        //--------------
        System.out.println("===========");
        for(Gender gender : Gender.values()){
            System.out.println(gender.value);
        }
       
        //--------------
        System.out.println("===========");
        for(OrderState order : OrderState.values()){
            System.out.println(order.getName());
        }
    }
    
}
```

​	