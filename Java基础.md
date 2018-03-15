1. 存在继承的情况下，代码执行顺序： 父类（静态变量、静态初始化块）-> 子类（静态变量、静态初始化块） -> 父类（变量、初始化块）-> 父类（构造器）-> 子类（变量、初始化块）-> 子类（构造器）
2. String, StringBuilder, StringBuffer, CharSequence。编译器会使用常量池保存那些在编译期就已经确定的常量。
    * CharSequence：表示一个字符序列，是一个接口。String, StringBuilder, StringBuffer都实现了它。
    * String：final修饰的不可变类型，所以线程安全。
    * StringBuilder：可变对象，未同步，不是线程安全的，不支持高并发情况，适用于单线程情况。效率很高，但不是线程安全的。
    * StringBuffer：可变对象，使用synchronized同步，线程安全，支持高并发情况，适用于多线程情况。效率低于StringBuilder。
    * string使用+相加的过程，JVM会先创建一个StringBuilder对象，通过StringBuilder.append()方法将str3与str4的值拼接，然后通过StringBuilder.toString()返回一个String对象赋值
    * 当通过语句str.intern()调用intern()方法后，JVM 就会在当前类的常量池中查找是否存在与str等值的String，若存在则直接返回常量池中相应String的引用；若不存在，则会在常量池中创建一个等值的String，然后返回这个String在常量池中的引用。因此，只要是等值的String对象，使用intern()方法返回的都是常量池中同一个String引用，所以，这些等值的String对象通过intern()后使用==是可以匹配的。
3. 整型的对象池缓存范围是-128到127，在这个范围内的整型引用都相等，因为对象池入池之后，再引用相同的值会取出同一个引用。如果超过这个范围，值相等，应用也不相等。valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接使用缓存池的内容。
4. switch可以接受 byte，short，char，int，从1.7开始支持string类型。
5. 静态方法在类加载的时候就存在了，它不依赖于任何实例，所以 static 方法必须实现，也就是说他不能是抽象方法 abstract。
6. try-catch-finally执行顺序
    * try-catch-finally都有return语句时，没有异常时，返回值是finally中的return返回的。try中的return语句包含运算的话也会进行计算，但是不返回main方法。finally中如果有return的话，程序将在这里返回main函数，返回操作之后的结果。如果没有的话，返回try之后再返回main函数，值不受finally影响，返回进入finally之前计算的值。
    * try-catch都有return语句时，没有异常时，返回值是try中的return返回的，不会因为finally对变量操作导致返回值改变。
    * try块中抛出异常，try、catch和finally中都有return语句，返回值是finally中的return。
    * 抛出异常后，执行catch块，执行完finally语句后，依旧返回catch中的执行return语句后的值，而不是finally中修改的值。
    * try块中出现异常到catch，catch中出现异常到finally，finally中执行到return语句返回，不检查异常。返回finally中return值。

7. 泛型提供了编译时的类型检测机制，该机制允许程序员在编译时检测到非法的类型。编译完成后会进行类型擦除。
8. 序列化：将对象转换成字节序列，方便存储和传输
    * Parcelable：手动编写序列化过程代码，以二级制方式写入，严重依赖于写入顺序，用户内存之中的序列化传输。
    * Serializable：通过标记，最终调用反射完成，产生大量临时变量，频繁引起GC。通常用户持久化到磁盘。
9. 字符串转换成integer原理：遍历字符串的字符数组，使用字符减去'0'转换成对象的数字，检测大小是否正常。再根据位数乘以10的幂。
10. java内部类:为了实现多继承。外部类继承一个类后，内部类在不改变外部类任何原本实现的情况下再继承另外一个类。获取被继承类的功能同时可以随意访问外部类，从而获取到两个类的功能和变量。
11. 静态类无法被重写：非静态方法在运行时才动态形成栈帧压入栈里进行类的绑定，静态方法在类加载的时候就已经绑定到类了，只与类有关，所以又称类方法，子类实例无法进行复写。

