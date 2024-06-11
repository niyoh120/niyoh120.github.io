## 类型

### 变量

变量使用let定义，常量使用const定义，变量默认是immutable的，如果需要声明一个mutable的变量，使用let mut

可以多次使用let定义名字相同的变量（变量的类型可以不同），后定义的变量会遮蔽掉前一个变量，以减少取名的麻烦

编译器在大部分情况下会根据上下文自动推倒变量的类型，但在有可能出现二义性的情况下，必须显式声明变量的类型，`let v:T`，可以声明部分类型，让编译器去推导剩下的类型

数字类型有i8,i16,i32,i64,isize，及对应的无符号类型，u前缀，一般使用i32或u32，索引使用usize

### 类型别名

```
type Age = u32;//可以是泛型type Double<T> = (T,Vec<T>)
```

### 全局变量

用static关键字声明，生命周期是’staitc，是整个程序执行期间

- 全局变量必须在声明的时候马上初始化
- 全局变量的初始化值必须是编译期可确定的常量，不能包括运行时才能确定的表达式、语句和函数调用(const fn是可以的)
- 带有mut修饰的全局变量，在使用的时候必须使用unsafe关键字

### 常量

用const声明,不是变量,初始化的值必须是编译期常量,没有模式匹配的效果

## 基本数据类型

### tuple和array

可以通过pattern match解构元组，也可以使用t.index来访问其中的元素，如t.0,t.1

array的长度是固定的，越界访问会panic

### 函数

函数的定义

```
fn function<'a,T>(x:&'a T)->&'a T{}
```

参数的类型必须显式声明

### 语句(statement)和表达式(expression)的区别

语句是一条指令，实现一个动作，以分号结尾，没有结果值 表达式求值得出一个结果值 表达式和语句的区分是Rust和一些语言（如C）的区别之一

函数由一系列语句和可选的表示返回值的表达式组成

### Struct

```
//可以定义一个tuple structstruct Foo(i32,i32);//然后这样使用let foo = Foo(42,49);println!("{}.{}",foo.0,foo.1);
```

### 循环

```
    while C{      }    loop{ //无限循环    }    for E in C{    }
```

```
fn main() {    //Like range in python    for i in 1..4 {        println!("{}!", i);    }    println!("done!");}
```

### ref 和 &

```
// 下面两行是等价的,a的类型都是&i32,一般情况下都用&，具体看http://xion.io/post/code/rust-patterns-ref.htmllet ref a = 2;let a = &2;
```

要理解两者的区别，需要理解什么是模式匹配，&会成为模式的一部分，ref不会，所以

```
fn main() {    let s = String::from("hello");    // let &sf = &s; 这句不能通过编译，sf的类型推导为String，sf会通过&s取得s的所有权，无法通过brrow check    let ref sf = &s; //可以编译，ref不影响pattern，sf的类型为 &&s}
```

引用类型要配合生命周期使用

### 模式匹配

看[[这篇文章](http://xion.io/post/code/rust-patterns-ref.html)](http://xion.io/post/code/rust-patterns-ref.html)

### 模块化

每个文件作为一个模块，或者文件夹为模块名，其中的mod.rs为模块入口，or mod’s namespace in single file. mod can be nested

### case need to annotate types annotate

use `collect` to return a container

### closure

infer capture value according to function use in closure, priority is &T, &mut T and T Using move before vertical pipes forces closure to take ownership of captured variables

the compiler will capture variables in the least restrictive manner possible. For instance, consider a parameter annotated as FnOnce. This specifies that the closure may capture by &T, &mut T or T, but the compiler will ultimately choose based on how the captured variables are used in the closure. 换句话说，每个closure至少都会实现FnOnce(表示最多可调用一次），call方法的第一个参数为self，如果该closure最多只有对外部环境的&mut访问，则会增加实现FnMut，call方法的第一个参数为&mut sefl，如果只有对外部环境的&访问，则实现Fn,表示该函数可重复调用且无副作用（也就是纯函数）,call方法的第一个参数为&self

Fn系列trait包含函数参数和返回值，比如FnOnce(E)->F 表示接收E类型的参数返回F类型的可调用一次的函数,FnOnce()表示函数没有参数也没有返回值

接受一个匿名函数为参数的函数必须声明泛型参数，调用时编译器会根据传入的匿名函数类型自动推导该参数

Diverging functions never return. They are marked using !, which is an empty type.

### Associated types

[[看这篇文章的](https://doc.rust-lang.org/rust-by-example/generics/assoc_items/types.html)](https://doc.rust-lang.org/rust-by-example/generics/assoc_items/types.html)

### 迭代器

调用next方法返回Option，结尾处返回None 迭代器本身是mutable的，不需要显式声明，因此只能使用一次 iter返回的是元素的inmutable reference iter_mut返回mutable reference into_iter move ownership to return value 迭代器的某些方法会消费这个迭代器（转移这个迭代器的所有权），比如sum方法，调用这类方法后迭代器就无效了（类似值被移走了） 另一些方法称作迭代器适配器，比如map，将某一函数作用在迭代器的所有元素上，这些函数是lazy的，需要在最后调用consumed方法才会执行，比如collent方法

实现 Iterator trait只需要实现next方法

### 智能指针

四种智能指针

- `Box<T>` 对应C++中的`unique_ptr`
- `Rc<T>`和`RefCell<T>`对应C++中的`shared_ptr`，区别是前者只能返回inmutable引用，后者可以返回inmutable或mutable引用
- `Weak<T>`对应C++中的`weak_ptr`
- `Arc<T>`是线程安全的`RC<T>`(Atomically Reference Counted)，只能返回immutable引用，如果需要修改状态，可以在其中装`Mutex`,`Rwlock`或者原子类型