# Rust语言及其Trait

今天我们来聊聊Rust中的Trait. 首先我们来理解一个概念: 抽象.

抽象就是把复杂的东西简单化。
假如现在这间屋子里面有两个中国人在掐架，一个白色衣服，一个红色衣服，那你是不是会表述成：一个白色衣服的人在跟一个红色衣服的人打架？再假如现在红色衣服的人被打跑了，来了个蓝色衣服的外国人，那你是不是会表述成：一个中国人在跟一个外国人打架？再假如现在蓝色衣服的人也被打跑了，来了个绿色的火星人，那你是不是会表述成：一个地球人在跟一个火星人打架？这个过程就是在不断抽象，将一个复杂的具体的东西，逐步简单化，使其更具通用性。

现在我说，看有一匹马！那你是不是在脑海中快速构建了一匹马的形象？然后再去寻找马在哪里？既然你都没有看见那匹马，你是怎么构建出来的形象呢？你构建的为何不会是一头猪或者一个人的形象。这就是抽象的能力，能够快速构建出某一类东西。而这个时候，你可以通过看到马的颜色和大小来完善你脑海中的那匹马，而不是你先想到颜色和大小才去找这匹马？因为我只会说看马？而不会先把马的照片给你看一眼，然后你再去寻找。
我们将其写成代码来看看？

``` rust

#![allow(dead_code)]

trait See {
    fn see(color: Color, age: u8) -> Self;
}

#[derive(Debug)]
enum Color {
    Red,
    White,
    Blue,
}

struct Horse {
    color: Color,
    age: u8,
}

impl See for Horse {
    fn see(color: Color, age: u8) -> Self {
        Horse { color, age }
    }
}

fn main() {
    let horse = Horse::see(Color::Red, 2);
    println!("Horse color: {:?}, age: {}", horse.color, horse.age);
}


```
这个例子中，trait代表了某种特征，然后不同的对象均可以实现该特征，而具有该特征的调用方法。

我们再深入一些, 看一下何为泛型约束.

我们来看一个对老师的定义：师者传道授业解惑也。我们来看定义一个老师，那么他必须能够传道授业和解惑，但是对于其他的，我们并不需要关注，比如他是小学的老师啊，还是孔子的三人行，必有我师的老师，只要他具有师者的定义，那么他就有传道授业和解惑这两个属性。而我们跟一个自称老师的人交流的时候，那么就可以直接向他提出疑惑，让他帮忙解惑，而不必关心他具体是几年级的老师，听着好像有些认知啊，你都不知道他是啥老师，你怎么问问题呢？其他不必担心，我们来看代码描述。

``` rust
type Answer = String;

enum Question {
    Language(String),
    Math(String),
    English(String),
}

trait Teacher {
    fn ask(&self, q: Question) -> Option<Answer>;
}

struct LanguageTeacher;

impl Teacher for LanguageTeacher {
    fn ask(&self, q: Question) -> Option<Answer> {
        match q {
            Question::Language(s) => Some(s),
            _ => None,
        }
    }
}

struct MathTeacher;

impl Teacher for MathTeacher {
    fn ask(&self, q: Question) -> Option<Answer> {
        match q {
            Question::Math(s) => Some(s),
            _ => None,
        }
    }
}

struct EnglishTeacher;

impl Teacher for EnglishTeacher {
    fn ask(&self, q: Question) -> Option<Answer> {
        match q {
            Question::English(s) => Some(s),
            _ => None,
        }
    }
}

fn ask1<T: Teacher>(q: Question, teacher: T) -> Option<Answer> {
    teacher.ask(q)
}

fn ask2(q: Question, teacher: impl Teacher) -> Option<Answer> {
    teacher.ask(q)
}

fn ask3(q: Question, teacher: Box<dyn Teacher>) -> Option<Answer> {
    teacher.ask(q)
}

fn main() {
    let answer1 = ask1(Question::Language("语文".to_owned()), LanguageTeacher);
    let answer2 = ask2(Question::Math("1+1".to_owned()), MathTeacher);
    let answer3 = ask3(Question::English("code".to_owned()), MathTeacher);
}

```
这里面有三个 ask 函数, 第一个是用泛型的方式, 指定传入的参数应该具有某者特性, 第二和第三种, 直接用特性作为参数, 在调用的时候, 直接传入具有该特性的对象. 前两种属于静态分发, 意思就是在编译期就完成了泛型的对象的确定, 并对此优化了性能, 而第三种是用Box来将置于堆上的对象, 在调用期间才会进行确定, 属于动态分发, 性能会略输一筹, 但是通用性和写法会更简洁一些.

我们接着再用面向对象的思维再来深入学习一下.
最简单的理解，还是拿我们人类来举例，假如我同时继承了爸爸和妈妈的一些特点，比如我理财的能力继承自我的父亲，然后我的美貌继承自我的母亲，我们现在来代码化我们三。

``` rust
trait Father {
    fn money(input: u32) -> u32 {
        input - 1
    }
}

trait Mother {
    fn age(&self) -> u32 {
        18 // comment: always
    }
}

```
现在爸妈的传承都已经准备就绪, 就看孩子还不能接下了.

``` rust
struct Son {
    age: u32,
}

impl Father for Son {}

impl Mother for Son {}

fn main() {
    let me = Son { age: 20 };
    assert_eq!(9, Son::money(10));
    assert_eq!(18, me.age());
}
```
这个孩子不负众望, 同时继承了爸妈的能力, 能够同时具有理财和魅惑众生的能力了.

可是这个时候, 妈妈发现, 哎呀, 你这个死鬼, 竟然敢藏私房钱, 拿来, 给老娘!

``` rust
trait Mother {
    fn money(input: u32) -> u32 {
        input + 1
    }
}

// Error: multiple application items in scope
// multiple "money" found
```

从此妈妈开始了理财生活, 但是呢, 爸爸依然贼心不改, 以为妈妈和他没有关联, 她就发现不了, 唉? 结果儿子想要继承的时候, 发现了, 冲突了, 继承了一样的方法, 编译器已经不让通过了, 爸爸悔之晚矣.

这个时候, 孩子说, 算了, 你们也别争了, 我自己来.

``` rust
impl Father for Son {
    fn money(input: u32) -> u32 {
        input + 1
    }
}

assert_eq!(11, Son::money(10));

```
这个就叫做重载, 重新实现了 "money" 方法, 再也不用担心爸爸妈妈的不合了.

但是这个时候, 爸爸不领情啊, 非得说, 我啥都能理, 不仅是钱, 好, 我们满足他!

``` rust
trait Father {
    type Money;

    fn money(input: Self::Money) -> Self::Money;
}

struct RMB(u32);

impl Father for Son {
    type Money = RMB;

    fn money(input: Self::Money) -> Self::Money { RMB(input.0 - 1) }
}

```
我们在其中设置了一个 关联类型, 用 `type` 来定义这个类型, 然后用 `Self::Money` 来代指那个类型. 这样孩子在继承的时候, 就需要给出 Money 的具体类型, 从而实现和使用.

这时候, 爸爸终于争了口气, 然后又想要更多, 我还想在理财的时候, 偷偷看一眼, 你理的具体是啥? 那我们肯定也得满足啊.

``` rust
use std::fmt::Debug;

trait Father {
    type Money: Debug;

    fn money(input: Self::Money)
        -> Self::Money
    {
        println!("{:?}", input);
        input
    }
}

#[derive(Debug)]
struct RMB(u32);


impl Father for Son {
    type Money = RMB;
}

```
看孝顺的孩子又给他实现了. 只需要爸爸声明 Money 的时候, 指定具有 Debug 特性, 就可以直接调用了.
这个烦人的爸爸有说了: 我学会了一种新理财方法, 不管给我啥, 我都能double一下.

```rust
use std::ops::Add;

trait Father {
    type Money: Add<Output = Self::Money> + Copy;

    fn money(input: Self::Money) -> Self::Money {
        input + input
    }
}
```
直接相加肯定是不行的, 所以我们给 Money 指定一个可以相加的特性, 这就需要, 在实现的时候, 该 Money 必须能够具有 Add 的特性

``` rust
#[derive(Eq, PartialEq, Ord, PartialOrd, Debug, Copy, Clone)]
struct RMB(u32);

impl Add for RMB {
    type Output = Self;

    fn add(self, other: RMB) -> Self {
        RMB(self.0 + other.0)
    }
}
```
看孩子在继承的时候, 所用的RMB就具有了相加的能力. 这个时候, 就自然满足了爸爸的继承要求.
但是生在国内, 我们还是希望能默认用人民币来理财, 所以我们给 Money 类型设置默认值.

``` rust
#![feature(associated_type_defaults)]

trait Father {
    type Money = RMB;

    fn money(input: RMB) -> RMB {
        input + input
    }
}

impl Father for Son {}
```
这个时候, 关联类型的默认值是只在nightly 版本中才会支持的, 而且需要设置feature, 使其生效.

好了, 到这边, 我们相亲相爱的一家人就结束了. Rust 中的 trait 博大精深, 各种用法层出不穷, 但是万法不离其宗. 大家用起来!
