### Stirng，StringBuffer，StringBuilder的区别

**1.可变性**

* String：不可变。对字符串操作的时候，实际上还是创建了一个新的字符串对象。
* StringBuffer和StringBuilder：可变，可以在原有对象进行修改，不会创建新的对象

**2.线程安全性**

* String中对象不可变，可以理解为常量，线程安全。

* StringBuffer：安全，包含同步方法，适合在多线程环境中使用
* StringBuilder：不安全，不包含同步方法，适合在单线程环境使用。

**3.性能**

* String：由于不可变，对字符串的任何修改都会导致创建新的字符串对象，会有额外的开销。
* StringBuffer：因为线程安全，包含同步方法，所以性能比StringBuilder慢一些
* StringBuilder：不包含同步方法，所以性能好一些。

**4.使用场景**

* String：适用于不经常修改的字符串 ，比如字符串常量。
* StringBuffer：适用于在**多线程环境中频繁**修改字符串的情况。
* StringBuilder：适用于在**单线程环境中频繁**修改字符串的情况。

