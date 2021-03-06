1. C和C++指针的最重要的区别在于：**C++是一种类型要求更强的语言。**就`void *`而言，这一点表现得更加突出。C虽然不允许随便地把一个类型的指针指派给另一个类型，但允许通过`void *`来实现。例如： 
    ```c
    bird* b;
    rock* r;
    void* v;
    v = r;
    b = v;
    ```
C++不允许这样做，其编译器将会给出一个出错信息。如果真的想这样做，必须显式地使用映射，通知编译器和读者。

2. 参数传递准则
当给函数传递参数时，人们习惯上应该是通过常量引用来传递，这种简单习惯可以大大提高效率：**传值方式需要调用构造函数和析构函数，然而如果不想改变参数，则可通过常量引用传递，它仅需要将地址压栈。** 事实上，只有一种情况不适合用传递地址方式，这就是当传值是唯一安全的途径，否则将会破坏对象（而不是修改外部对象，这不是调用者通常期望的）。

3. C++访问权限控制：public、private、protected
其中protected只有在继承中才有不同含义，否则与private相同，也就是说**两者只有一点不同：继承的结构可以访问protected成员，但不能访问private成员。**

4. 前置声明注意
    ```cpp
    struct X;  // Declaration(incomplete type spec)
    struct Y
    {
      void f(X *memx);  
      void g(X memx);  // not allowed, the size of X is unknown.
    };
    ```
这里f(X\*)引用了一个X对象的地址，这是没有任何问题的，但如果是`void g(X memx);`就不行了，编译器会报错。这一点很关键，因为**编译器知道如何传递一个地址，这一地址大小是一定的，而不用管被传递的对象类型大小。如果试图传递整个对象，编译器就必须知道X的全部定义以确定它的大小以及如何传递它，这就使程序员无法声明一个类似于Y :: g(X) 的函数。**

5. C++是纯的吗？
如果某个类的一个函数被声明为`friend`，就意味着它不是这个类的成员函数，但却可以修改类的私有成员， 而且它必须被列在类的定义中，因此我们可以认为它是一个特权函数。这种类的定义提供了有关权限的信息，我们可以知道哪些函数可以改变类的私有部分。 因此，**C++不是完全的面向对象语言，它只是一个混合产品。`friend`关键字就是用来解决部分的突发问题。它也说明了这种语言是不纯的。毕竟C + +语言的设计是为了实用，而不是追求理想的抽象。**

6. C++输入输出流的操纵算子（manipulator）有：endl、flush、ws、hex等。
    ```cpp
    cout<<flush;   // 清空流   
    cout << hex << "0x" << i;  // 输出16进制   
    cin>>ws;  // 跳过空格
    ```
iostream.h还包括以下的操纵算子：
    ![操作算子符号表](http://upload-images.jianshu.io/upload_images/46178-a1561e83478c9a76.png)
 **如何建立我们自己的操纵算子？**
我们可能想建立自己的操纵算子，这是相当简单的。**一个像endl这样的不带参数的操纵算子只是一个函数，这个函数把一个ostream引用作为它的参数**。对endl的声明是： 
    ```cpp
    ostream& endl(ostream&)；
    ```
**例子**：产生一个换行而不刷新这个流。人们认为nl比使用endl要好，因为后者总是清空输出流，这可能引起执行故障。
    ```cpp
    ostream& nl(ostream& os) {
      return os << "\n";
    }
    int main() {
      cout << "newlines" << nl << "between" << nl << "each" << nl << "word" << nl;
     return 0;
    }
    ```

7. C语言中const与C++中const的区别：
常量引进是在早期的C++版本中，当时标准C规范正在制订。那时，常量被看作是一个好的思想而被包含在C中。但是，**C中的const意思是“一个不能被改变的普通变量”，在C中，它总是占用存储而且它的名字是全局符。C编译器不能把const看成一个编译期间的常量。**在C中， 如果写：
    ```cpp
    const bufsize=100；
    char buf[bufsize]；
    ```
尽管看起来好像做了一件合理的事，但这将得到一个错误结果。**因为bufsize占用存储的某个地方，所以C编译器不知道它在编译时的值**。在C语言中可以选择这样书写：
    ```cpp
    const bufsize；
    ```
这样写在C++中是不对的，而C编译器则把它作为一个声明，这个声明指明在别的地方有存储分配。因为**C默认const是外部连接的，C++默认cosnt是内部连接的**，这样，如果在C++中想完成与C中同样的事情，必须用extern把连接改成外部连接：
    ```cpp
    extern const bufsize;//declaration only
    ```
这种方法也可用在C语言中。
**注意：**在C语言中使用限定符const不是很有用，即使是在常数表达式里（必须在编译期间被求出）；想使用一个已命名的值，使用const也不是很有用的。C迫使程序员在预处理器里使用#define。

8. 类里的const和enum
下面的写法有什么问题吗？：
    ```cpp
    class bob {
        const size = 100;  // illegal
        int array[size];   // illegal
    }
    ```
结果当然是编译不通过。why？因为**const在类对象里进行了存储空间分配**，编译器不能知道const的内容是什么，所以不能把它用作编译期间的常量。**这意味着对于类里的常数表达式来说，const就像它在C中一样没有作用。**

 在类里的const意思是“在这个特定对象的寿命期内，而不是对于整个类来说，这个值是不变的”。那么怎样建立一个可以用在常数表达式里的类常量呢？
 **一个普通的办法是使用一个不带实例的无标记的enum**。枚举的所有值必须在编译时建立，它对类来说是局部的，但常数表达式能得到它的值，这样，我们一般会看到： 
    ```cpp
    class bob {
        enum { size = 100 };  // legal
        int array[size];      // legal
    }
    ```
**使用enum是不会占用对象中的存储空间的**，枚举常量在编译时被全部求值。我们也可以明确地建立枚举常量的值：`enum { one=1,two=2,three}；`

9. 类里面的const成员函数
    ```cpp
    class X {
        int i;
    public:
        int f() const;      
    }
    ```
这里f()是const成员函数，表示只能const类对象调用这个函数（const对象不能调用非const成员函数），如果我们改变对象中的任何一个成员或调用一个非const成员函数，编译器将发出一个出错信息。
**关键字const必须用同样的方式重复出现在定义里，否则编译器把它看成一个不同的函数：**
    ```cpp
    int X::f() const { return i；}
    ```
任何不修改成员数据的函数应该声明为const函数，这样它可以由const对象使用。
**注意：构造函数和析构函数都不是const成员函数，因为它们在初始化和清理时，总是对对象作些修改。**

 ----------
 ###引申：如何在const成员函数里修改成员 —— 按位和与按成员const###
如果我们想要建立一个const成员函数，但仍然想在对象里改变某些数据，这时该怎么办呢？这关系到按位const和按成员const的区别。**按位const意思是对象中的每个位是固定的，所以对象的每个位映像从不改变。按成员const意思是，虽然整个对象从概念上讲是不变的，但是某个成员可能有变化。**当编译器被告知一个对象是const对象时，它将保护这个对象。

 这里我们要介绍在const成员函数里改变数据成员的两种方法。 
 - 第一种方法已成为过去，称为“**强制转换const**”。它以相当奇怪的方式执行。取this（这个关键字产生当前对象的地址）并把它强制转换成指向当前类型对象的指针。看来this已经是我们所需的指针，但它是一个const指针，所以，还应把它强制转换成一个普通指针，这样就可以在运算中去掉常量性。下面是一个例子：
    ```cpp
    class Y {
      int i, j;
    public:
      Y() { i = j = 0; }
      void f() const;
    };
    
    void Y::f() const {
    //!  i++;  // error
        ((Y*)this)->j++;  // ok , cast away const feature.
    }
    ```
     这种方法可行，在过去的程序代码里可以看到这种用法，**但这不是首选的技术**。问题是：**this没有用const修饰，这在一个对象的成员函数里被隐藏**，这样，如果用户不能见到源代码（并找到用这种方法的地方），就不知道发生了什么。
 - 第二种方法也是推荐的方法，就是在类声明里使用关键字`mutable`，以指定一个特定的数据成员可以在一个const对象里被改变。
    ```cpp
    class Y {
      int i;
      mutable int j;
    public:
      Y() { i = j = 0; }
      void f() const;
    };
    
    void Y::f() const {
    //!  i++;  // error
        ((Y*)this)->j++;  // ok , mutable.
    }
    ```
    
10. volatile关键字
volatile的语法与const是一样的，但是volatile的意思是“在编译器认识的范围外，这个数据可以被改变”。不知何故，环境正在改变数据（可能通过多任务处理），所以，**volatile告诉编译器不要擅自做出有关数据的任何假定—在优化期间这是特别重要的。**如果编译器说：“我已经把数据读进寄存器，而且再没有与寄存器接触”。一般情况下，它不需要再读这个数据。但是，如果数据是volatile修饰的，编译器不能作出这样的假定，**因为可能被其他进程改变了， 它必须重读这个数据而不是优化这个代码。** 

 注意：
>- 就像建立const对象一样，程序员也可以建立volatile对象，甚至还可以建立const volatile对 象，这个对象不能被程序员改变，但可通过外面的工具改变。
>- 就像const一样，我们可以对数据成员、成员函数和对象本身使用volatile，可以并且也只能为volatile对象调用volatile成员函数。
>- volatile的语法与const是一样的，所以经常把它们俩放在一起讨论。为表示可以选择两个中的任何一个，它们俩通称为**c-v限定词**。

####接下来继续：[C++编程思想重点笔记（下）](http://blog.csdn.net/lanxuezaipiao/article/details/41673883)