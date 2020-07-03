# Android Room

## 返回查询结果 column 的集合

有些时候只需要查询数据库中的表的某些字段即可，比如只需要 User Entity 中的 first_name 和 last_name 两个字段，而不是所有的字段，只取需要的字段，而不是查询所有字段，会节约资源并且使你的查询更快。

Room 允许从查询返回任何基于 Java 的基本类型对象，只要查询结果的返回列中可以映射到返回对象。

```kotlin
data class NameTuple(
		@ColumnInfo(name = "first_name) val firstName:String?,
		@ColumnInfo(name = "last_name) val lastName:String?
)
```

此时，可以在查询方法中使用此对象

```kotlin
@Dao
interface MyDao {
		@Query("SELECT first_name, last_name FROM user")
  	fun loadFullName(): List<NameTuple>
}
```

## 直接返回 Cursor

```kotlin
@Dao
interface MyDao {
 	@Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
  fun loadRawUsersOlderThan(minAge: Int): Cursor
}
```

> 高度不建议直接使用 Cursor API，因为它无法保证 row 是否存在，或者 row 包含了什么值。除非是有现存的无法重构的代码，已经使用了 Cursor，否则不要直接使用。

