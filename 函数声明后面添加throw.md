## 函数声明后面添加 `throw()`

**异常安全，不允许抛出任何异常**

```
void fun() throw();
```

**可以抛出任何形式的异常**

```
void fun() throw(...);
```

**只能抛出特定类型的异常**

```
void fun() throw(exceptionType);
```
