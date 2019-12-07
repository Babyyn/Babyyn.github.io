# YAML

org.yaml.snakeyaml.Yaml



**yml**

```
!!cn.com.luckin.User
name: Lockin
age: 1000
```

**Bean**

```java
@Getter
@Setter
@ToString
public class User {

    private String name;

    private int age;
}
```



### Example

```java
public class UserTest {

    @Test
    public void userTest(){
        User ur = loadUser();
        System.out.println(ur.toString());
        // User(name=Starbucks, age=1000)
    }

    public static User loadUser() {
        return loadAs("fixtures/user/home-user.yml", User.class);
    }

    private static <T> T loadAs(String classPathResource, Class<T> type) {
        Yaml yaml = new Yaml(new JodaDateTimePropertyConstructor());
        yaml.setBeanAccess(BeanAccess.FIELD);
        try (InputStream fileStream = new ClassPathResource(classPathResource).getInputStream()) {
            return yaml.loadAs(fileStream, type);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private static class JodaDateTimePropertyConstructor extends Constructor {
        JodaDateTimePropertyConstructor() {
            yamlClassConstructors.put(NodeId.scalar, new TimeStampConstruct());
        }

        class TimeStampConstruct extends ConstructScalar {
            @Override
            public Object construct(Node nnode) {
                if ("tag:yaml.org,2002:timestamp".equals(nnode.getTag().getValue())) {
                    Construct dateConstructor = yamlConstructors.get(Tag.TIMESTAMP);
                    Date date = (Date) dateConstructor.construct(nnode);
                    return new DateTime(date);
                } else {
                    return super.construct(nnode);
                }
            }
        }
    }
}
```

