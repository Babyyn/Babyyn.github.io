# Kotlin性能优化

几类常见的Kotlin性能损耗点：

1. Companion Object-伴生对象
2. Higher-order functions and Lambda expressions-高阶函数和Lambda表达式
3. Local functions-本地函数
4. Null safety-空安全
5. Varargs-变长参数
6. Delegated properties-委托属性
7. Ranges
8. RecyclerView.ViewHolder

## Companion Object

#### Kotlin代码

```kotlin
class User {

    private var name: String = ""

    private var age: Int = 0

    companion object {
        fun getUser(): User {
            val ur = User()
            ur.age = 28
            ur.name = "Ccf"
            return ur
        }
    }
}
```

