# 主页

## 什么是Themis  
!!! note "低代码框架"
    Themis 是一个低代码框架负责将以下代码部分以低代码的方式实现
## 开发一个需求  

!!! note "在介绍Themis之前我们先来看一段常见的业务需求和如何实现"
    例如在前台下单前（知道用户、门店、金额）需要获取可以享受多少折扣  

- 业务规则是 
    1. 门店等级为甲级 或者 门店所属区域为京津冀区域 则 折扣为7折 .
    2. 否则 门店为新店 则 折扣为8折 ,但是开业多少天内的门店为新店采用公参配置.
    3. 否则 订单金额大于10000 折扣为9折.
    3. 均不满足则不享受折扣 .

    一个枚举类:Level表示门店等级 分别为10甲20乙30丙  
    两个实体类:Area表示区域、Store表示门店,其中门店中存在一个属性项area数据类型为区域对象  

```java linenums="1"
/**
 * 区域
 */
public class Area {
    //编码
    private String code;
    //区域名称
    private String name;
}
/**
 * 门店等级枚举
 */
public enum Level {
    L_10("10","甲"),
    L_20("20","乙"),
    L_30("30","丙");
    private String code;
    private String name;
    private Level(String code, String name) {
        this.code = code;
        this.name = name;
    }
}
/**
 * 门店
 */
public class Store {
    //编码
    private String code;
    //门店名称
    private String name;
    //开业时间
    private LocalDate startTime;
    //门店等级
    private Level level;
    //门店所属区域
    private Area area;
}
```


- 业务代码将可以采用如下实现
    * getDiscount 方法为获取折扣方法 入参 用户编码、门店编码、金额 返回值是具体折扣
    * getNewStoreDays 方法为获取新店天数的公参配置 返回值是具体新店天数
    * getDistanceNowDays 方法工具方法为获取一个时间到现在的天数  入参为时间  返回值为天数
    * getStore 方法为根据门店编码获取门店完整信息 入参门店编码 返回值 门店对象
    * getArea 方法为根据区域编码获取区域完整信息 入参区域编码 返回值 区域对象

```java linenums="1"
/**
 * 下单相关
 */
public class CreateOrder {
    
    public BigDecimal getDiscount(String userCode, String storeCode, BigDecimal amount) {
        if(storeCode==null||storeCode==''){
            return new BigDecimal("1");
        }
        Store store = getStore(storeCode);
        if ((store.level == Level.L_10) || (store.area.code == "JJJ")) {
            return new BigDecimal("0.7");
        }
        int day = getNewStoreDays();
        if (getDistanceNowDays(store.startTime) < day) {
            return new BigDecimal("0.8");
        } else if (amount>10000){
            return new BigDecimal("0.9");
        }

        return new BigDecimal("1");
    }

    private int getNewStoreDays() {
        return 365;
    }

    private int getDistanceNowDays(LocalDate history) {
        Period period = Period.between(history, LocalDate.now());
        return period.getDays();
    }

    private Store getStore(String storeCode) {
        Store store = new Store();
        store.code = "1002";
        store.name = "XX门店";
        store.level = Level.L_20;
        store.area = getArea("JJJ");
        return store;
    }

    private Area getArea(String areaCode) {
        Area area = new Area();
        area.code = "JJJ";
        area.name = "京津冀";
        return area;
    }
}
```



    

