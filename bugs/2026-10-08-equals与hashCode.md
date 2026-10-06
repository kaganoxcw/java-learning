# 2026-10-08 【HashSet 去重失效：只重写了 equals 没重写 hashCode】

- 出现场景：用 `HashSet<Student>` 给两个"学号相同"的学生对象去重
- 报错信息：无报错，**结果不符合预期**（set 里出现了 2 个元素）
- 定位耗时：约 40 分钟

---

## 一、现象

```java
Set<Student> set = new HashSet<>();
set.add(new Student("001", "张三"));
set.add(new Student("001", "张三"));
System.out.println(set.size());
```

期望输出 `1`，实际输出 `2`。

---

## 二、原因

`Student` 类只重写了 `equals`，没有重写 `hashCode`。

HashSet 的判定流程是：
1. 先算对象的 `hashCode`，定位到数组的哪个桶
2. 桶里再用 `equals` 比较

我只重写了 `equals`，两个对象的 `hashCode` 仍然是 Object 默认的
**内存地址派生值**，根本不同 → 直接被分到不同的桶 → 连 `equals` 都没机会执行。

---

## 三、解法

`equals` 和 `hashCode` 一起重写（IDEA 里用 `Alt + Insert` 自动生成）：

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Student student = (Student) o;
    return Objects.equals(id, student.id);
}

@Override
public int hashCode() {
    return Objects.hash(id);
}
```

改完后输出 `1`。

---

## 四、以后怎么避免

> **规则：只要用自定义对象当 HashMap 的 key 或放进 HashSet，第一件事就是检查 equals 和 hashCode 有没有一起重写。**

用 IDEA 自动生成时，选 `equals() and hashCode()` 一起生成，不要只选一个。

---

## 五、相关知识

- 涉及知识点：Object 的 equals / hashCode 契约、HashSet 去重原理
- 对应笔记：`notes/02-面向对象/Object的equals与hashCode.md`、`notes/04-集合/HashSet去重原理.md`
