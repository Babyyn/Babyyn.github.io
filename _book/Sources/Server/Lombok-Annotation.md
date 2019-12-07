# Lombok 注解标签

## Configuration

使用Lombok需要给Intellij IDEA安装Lombok插件。

### build.gralde

```groovy
implementation 'org.projectlombok:lombok:1.18.8'
annotationProcessor 'org.projectlombok:lombok:1.18.8'
```

### lombok.config

```groovy
clear config.stopBubbling
config.stopBubbling = true
clear lombok.addLombokGeneratedAnnotation
lombok.addLombokGeneratedAnnotation = true
```

> lombok.config 和 build.gradle 在同级目录下。



## Annotation

**Getter/Setter**: 给Java类生成get/set方法，适用于**Class**和**Field**。

**NonNull**: 空检测注解，适用于**Field**、**Method**、**Parameter**，如果相应的字段或者参数尝试设置为空，会报NullPointerException。

**NoArgsConstructor**: 生成无参数的构造方法。

```java
@NoArgsConstructor
public class Address {

    private String detail;

    public Address(String detail) {
        this.detail = detail;
    }
}
```

使用了NoArgsConstructor注解之后，注解会自动生成无参数的构造方法。

**RequiredArgsConstructor**: 生成一个包含必选Field参数的构造方法，定义为final并且有约束的Field(比如使用NonNull注解标记的Field)是必选Field。

```java
@RequiredArgsConstructor
public class Address {

    @NonNull
    private final String detail;
}
```

上面的类可以通过 `new Address("detail")` 创建一个实例。

**AllArgsConstructor**: 根据所有的Field生成构造方法。

```java
@AllArgsConstructor
public class Address2 {

    @NonNull
    private String detail;

    @NonNull
    private String phone;
}
```

上面的类可以通过 `new Address("detail", "110");` 创建一个实例。

**EqualsAndHashCode**: 为Bean类生成equals()方法和hashCode()方法，基于相关的field。

**Data**: 为Field生成getter/setter, toString(), hashCode(), equals(), Constructor方法。此注解等同于Getter, Setter, RequiredArgsConstructor, ToString, EqualsAndHashCode，适用于**Class**。

> `@Data(staticConstructor = "getInstance")` 生成静态方法getInstance()，获取一个类实例。

**Builder**: 自动生成Builder构造类，便于使用Builder模式构造实体，适用于**Class**。

```java
@Builder
@Getter
public class Address {

    @NonNull
    private String detail;

    @NonNull
    private String phoneNumber;
}
```

定义一个Address类，添加Builder注解，注解解释器自动为我们生成AddressBuilder类。

```java
Address address = Address.builder()
        .detail("ShanXi Xi'an")
        .phoneNumber("110")
        .build();
```

**自定义Builder**

上面注解生成的设置phoneNumber的方法，不会做phoneNumber是否有效的check，我们可以自定义Builder类，来check phoneNumber是否有效。

```java
@Builder
@Getter
public class Address {

    @NonNull
    private String detail;

    @NonNull
    private String phoneNumber;

    /**
     * customized builder.
     */
    public static class AddressBuilder {
        /**
         * Address builder.
         * 版本号会被校验，校验规则取自 Condition.PHONE_NUMBER_REGEX
         *
         * @param phoneNumber user input phoneNumber
         * @return this
         * @throws InvalidAppVersionException if phoneNumber is invalid
         */
        public AddressBuilder phoneNumber(String phoneNumber) {
            if (Objects.isNull(phoneNumber)
                    || !Pattern.compile(Condition.PHONE_NUMBER_REGEX).matcher(phoneNumber).matches()) {
                throw new InvalidPhoneNumberException(phoneNumber);
            }

            this.phoneNumber = phoneNumber;
            return this;
        }
    }
}
```

