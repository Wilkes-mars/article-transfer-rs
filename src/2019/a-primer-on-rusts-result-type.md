# 【译】Rust 的 Result 类型入门
>* A Primer on Rust’s Result Type 译文

>* 原文链接：https://medium.com/@JoeKreydt/a-primer-on-rusts-result-type-66363cf18e6a
>* 原文作者：[Joe Kreydt](https://medium.com/@JoeKreydt?)
>* 译文出处：https://github.com/suhanyujie/article-transfer-rs
>* 译者：[suhanyujie](https://www.github.com/suhanyujie)
>* tips：水平有限，翻译不当之处，还请指正，谢谢！

* ![](https://miro.medium.com/max/2099/1*AoZOz1AJS15yyB3TLUn93A.jpeg)
The Result type is the de facto way of handling errors in Rust, and it is clever; maybe too clever.
>`Result` 类型是 Rust 中处理错误的常用方法类型，它比较灵活；也许是非常灵活！

Result is not exactly intuitive for those learning Rust, and trying to figure out how to use it by reading [its documentation page is](https://doc.rust-lang.org/std/result/) a good way to turn your brain into a french fry. That’s fine if you’re hungry, but if you need to handle an error or use a function that returns a Result type (and many do), then it’s not cool.
>对于那些正在学 Rust 的人来讲，Result 可能不是那么直观，可以通过阅读它的[文档](https://doc.rust-lang.org/std/result/)来了解如何使用是个不错的方法。如果你想迫切的学会它，也是可以的，但如果你只是用它处理错误或者使用某个返回 Result 类型的函数（很多人都这样做），你可能体会不到它的妙处。

In hopes of saving a few brains, I am going to explain Rust’s Result in plain English.
>为了节省大家的时间，我打算使用英语来解释 Rust 的 Result 类型。

# Result 是什么？
## According to [the Rust book](https://doc.rust-lang.org/1.30.0/book/first-edition/error-handling.html)
>参考[Rust 编程手册](https://doc.rust-lang.org/1.30.0/book/first-edition/error-handling.html)

“Result expresses the possibility of error. Usually, the error is used to explain why the execution of some computation failed.”
>“Result 表达的是错误的可能性。通常错误是用来解释某种任务执行失败的原因。”

![](https://miro.medium.com/max/400/1*g1A-DkLZ6dPjOKTo4kzrGg.gif)

## In Plain English
>用英语简单的解释

Result is a type returned by a function that can be either Ok or Err. If it is Ok, then the function completed as expected. If it is Err, then the function resulted in an error.
>Result 是一个函数返回的类型，它可以是 Err，也可以是 Err。如果是 Ok，则表示函数按照预期执行完成。如果是 Err，则该函数出现了错误。

# What does Result do? 
>Result 用来做什么？

## According to the Rust book 
>根据 Rust 编程手册

“The Result type is a way of representing one of two possible outcomes in a computation. By convention, one outcome is meant to be expected or “Ok” while the other outcome is meant to be unexpected or “Err”.”
>Result 类型是对计算过程中可能出现的结果的表示方式。按照惯例，如果一个结果是预期的 `Ok`，那么另一个结果则是意料之外的，即 `Err`。

## In English, Please
Functions return values. Those values are of a certain data type. A function can return a value of the Result type. The Result type changes based on whether the function executed as expected or not. Then, the programmer can write some code that will do A if the function executed as expected, or B if the function encountered a problem.
>函数返回值。这些值具有特定的数据类型。函数可以返回 Result 类型的结果。Result 类型根据函数是否按预期执行而变化。然后，程序员可以编写一些代码，在函数按预期执行 A，

# Error If You Don’t Handle a Result
>不处理 Result，则产生异常

```
error[E0308]: mismatched types
  --> main.rs:20:26
   |
20 |     let my_number: f64 = my_string.trim().parse(); //.unwrap();
   |                          ^^^^^^^^^^^^^^^^^^^^^^^^ expected f64, found enum `std::result::Result`
   |
   = note: expected type `f64`
              found type `std::result::Result<_, _>`
error: aborting due to previous error
For more information about this error, try `rustc --explain E0308`.
compiler exit status 1
```

The key part of the error is, “expected f64, found enum.” In similar scenarios, you might also see:
>报错信息中关键的部分是，“expected f64, found enum.”，类似的场景中，可能还会有：

    - “expected u32, found enum”
    - “expected String, found enum”
    - “expected [insert type here], found enum”

**If you get an error like the ones above, it is because you need to handle a Result that has been returned by a function in your program.**
>如果你得到一个类似上面的错误，那是因为需要你处理函数返回的 Result 类型数据

# A Program that Results in the Error
>程序报错

```rust
use std::io::{stdin, self, Write};
fn main(){
    let mut my_string = String::new();
    print!(“Enter a number: “);
    io::stdout().flush().unwrap();
    stdin().read_line(&mut my_string)
        .expect(“Did not enter a correct string”);
    let my_number: f64 = my_string.trim().parse();
    println!(“Yay! You entered a number. It was {:?}”, my_num);
}
```

In this program, the user is prompted to enter a number. Then, the user’s input is read and stored as a string type. We want a number type, not a string, so we use the _parse()_ function to convert the string to a 64 bit floating point number (f64).
>在这个程序中，它提示用户输入一个数字。然后将输入作为字符串读入并存储下来。我们想要的是一个数值类型，不是 String，所以我们需要使用 _parse()_ 函数将其转换为一个 64 位浮点数（f64）。

If the user enters a number when prompted, the _parse()_ function should have no issues converting the user’s input string into f64. Yet we get an error.
>如果用户输入的是一个数字，那么 _parse()_ 函数将其转换为 f64 没什么大问题。但我们仍然会得到一个错误。

The error occurs because the _parse()_ function does not just take the string, convert it to a number, and return the number that it converted. Instead, it takes the string, converts it to a number, then returns a Result type. The Result type needs to be unwrapped to get the converted number.
>发生错误是因为 _parse()_ 函数不只是将 String 转换为数字并返回。相反，它接受字符串，将其转换为数字，然后返回 Result 类型。Result 类型需要被解包才能得到我们需要的数值。

## Fixing the Error with Unwrap() or Expect()
>用 Unwrap() 或 Expect() 修复错误

The converted number can be extracted from the Result type by appending the _unwrap()_ function after the _parse()_ like so:
>转换后的数字可以可以通过在 _parse()_ 后面附加调用 _unwrap()_ 函数将数字从 Result 中“解压”出来，类似于这样： 

```rust
let my_number: f64 = my_string.trim().parse().unwrap();
```

The _unwrap()_ function looks at the Result type, which is either _Ok_ or _Err_. If the Result is _Ok_, _unwrap()_ returns the converted value. If the Result is _Err_, _unwrap()_ lets the program crash.
>_unwrap()_ 函数可以看出 Result 中类型，可能是 _Ok_，也可能是 _Err_。如果 Result 中包裹的类型是 _Ok_，那么 _unwrap()_ 则返回它的值。如果 Result 中的类型是 _Err_，_unwrap()_ 则会让程序崩溃。

![](https://miro.medium.com/max/470/1*bPYM5NAZ8OYenRAejcI7uA.gif)

You can also handle a Result with the _expect()_ function like so:
>你也可以用 _expect()_ 函数像下方这样来处理 Result：

```rust
let my_number: f64 = my_string.trim().parse().expect(“Parse failed”);
```

The way _expect()_ works is similar to _unwrap()_, but if the Result is _Err_, _expect()_ will let the program crash **and** display the string that was passed to it. In the example, that string is “Parse failed.”
>_expect()_ 的工作方式类似于 _unwrap()_，假如 Result 是 _Err_，_expect()_ 将会使程序崩溃**并且**将其中的字符串内容——"Parse failed."展示在标准输出中。

## The Problem with Unwrap() and Expect()
>使用 unwrap() 和 expect() 的缺陷

When using the _unwrap()_ and _expect()_ functions, the program crashes if there is an error. That is okay when there is very little chance of an error occurring, but in some cases an error is more likely to occur.
>当我们使用 _unwrap()_ 和 _expect()_ 函数时，如果遇到错误，程序会发生崩溃。如果错误发生的几率非常小，这也许可以容忍，但在某些情况下，错误发生的概率会比较大。

In our example, it is likely that the user will make a typo, entering something other than a number (maybe a letter or a symbol). We do not want the program to crash every time the user makes a mistake. Instead, we want to tell the user to try entering the number again. This is where Result becomes very useful, especially when it is combined with a match expression.
>在上面的示例中，用户可能输入错误，输入的不是数值（可能是字母或者特殊符号）。我们并不想每次用户输入错误的内容程序就发生崩溃。相反，我们应该提示用户应该输入数字。这种场景下，Result 就非常有用，尤其是当它与一个匹配表达式相结合的时候。

# Fixing the Error with a Match Expression
>用匹配表达式修复错误

```rust
use std::io::{stdin, self, Write};
fn main(){
    let mut my_string = String::new();
    print!(“Enter a number: “);
    io::stdout().flush().unwrap();
    let my_num = loop {
        my_string.clear();
        stdin().read_line(&mut my_string)
            .expect(“Did not enter a correct string”);
        match my_string.trim().parse::<f64>() {
            Ok(_s) => break _s,
            Err(_err) => println!(“Try again. Enter a number.”)
        }
    };
    println!(“You entered {:?}”, my_num);
}
```

If you ask me, that is some fun code!
>如果你问我怎么实现，上面就是示例代码！

The main difference between the fixed program and the broken program above is inside the loop. Let’s break it down.
>前面提到的不优雅的实现和优雅的实现方式的不同点是在循环体内部。我们可以分解一下。

# The Code Explained
>代码分析

Before the loop, we prompted the user to enter a number. Then we declared my_num.
>在 loop 之前，我们提示用户输入一个数字。接着我们声明 my_num。

We assign the value returned from the loop (the user’s input, which will have been converted from a string to a number) to my_num:
>我们将循环体中返回的值（用户的输入，它将从字符串转换为数字）赋给 my_num：

```rust
let my_num = loop {
```

Within the loop, we handle the user’s input. After we accept the input, there are three problems we need to solve.
>在循环体中，我们阻塞等待用户输入。然后接收用户的输入，在这个过程中我们有三个问题要解决。

- 1.We need to make sure the user entered a number and not something else, like a word or a letter.
- 1.我们需要确定用户输入的是数字而非其他的字符，一个词或者一个字母。
- 2.Rust’s _read_line()_ function takes the user’s input as a string. We need to convert that string to a floating-point number.
- 2.Rust 中的 _read_line()_ 函数可以以字符串的类型拿到用户的输入。我们需要将其转换为浮点数。
- 3.If the user doesn’t enter a number, we need to clear out the variable that holds the user’s input and prompt the user to try again.
- 3.如果用户没有输入数字，我们需要清理变量，并提示和等待用户再次输入。


Part of problem number three (clearing the my_string variable) is solved in the first line inside the loop:
>在第三部分问题（清理 my_string 变量）在循环体内的第一行就已经实现了：

```rust
my_string.clear();
```

Next, we accept the user’s input with:
>下一步，我们接收用户的输入：

```rust
stdin().read_line(&mut my_string)
    .expect(“Did not enter a correct string”);
```

The _read_line()_ function returns a Result type. We are handling that Result with the _expect()_ function. That is fine in this situation because the chances of _read_line()_ resulting in an error is extremely rare. A user typically cannot enter anything but a string in the terminal, and that is all _read_line()_ needs in this case.
>_read_line()_ 函数返回一个 Result 类型。我们使用 _expect()_ 函数处理它。在这种情形下是完全没问题的，因为 _read_line()_ 出错的几率非常小。用户通常只能在终端输入一个字符串，而这正是 _read_line()_ 所需要的。

The string of user input returned by _read_line()_ is stored in the _my_string_ variable.
>通过 _read_line()_ 把用户输入的字符串返回并存在 _my_string_ 变量中。

## The Juicy Part
>渐入佳境

Now that we have the user’s input stored in my_string, we need to convert the string to a floating-point number. The _parse()_ function does that conversion, then returns the value in a Result. So we have more Result handling to do, but this time there is a good chance we will have an error on our hands. If the user enters anything but a number, _parse()_ will return an error Result (_Err_). If that happens, we don’t want the program to crash. We want to tell the user that a number wasn’t entered, and to try again. For that, we need our code to do one thing if _parse()_ succeed and something else if _parse()_ failed. Queue the match expression.
>现在我们已经将输入的字符串存在 _my_string_ 中，我们需要将其转换为浮点数。使用 _parse()_ 函数可以实现，然后将浮点数结果返回。所以我们有不止 Result 的类型需要处理，但这一次，我们很可能会出现一个错误。如果用户输入的是非数字， _parse()_ 将会返回一个错误类型的 Result（_Err_）。如果发生这种情况，我们不希望程序崩溃。而是希望提示用户没有输入正确的数字，请再试一次。为此，我们需要写好调用 _parse()_ 成功时的逻辑，还要写好调用失败时的逻辑。类似于逐个处理匹配表达式可能的结果。

## Dissecting the Match Expression
>分析匹配表达式

```rust
match my_string.trim().parse::<f64>() {
    Ok(_s) => break _s,
    Err(_err) => println!(“Try again. Enter a number.”)
}
```

First, we use the match keyword to declare our match expression. Then, we provide the value that will be analyzed by the match expression. That value comes from:
>首先，我们使用 match 关键字来声明匹配表达式。然后，我们提供与表达式匹配的分析值。这个值就是下面所示：

```rust
my_string.trim().parse::<f64>()
```

That little chunk of code takes my_string, which is holding the user’s input, and feeds it into the _trim()_ function. _Trim()_ just removes any extra lines or spaces that might have tagged along with the user’s input. We need _trim()_ because _read_line()_ attaches an extra line to the input, which messes up the type conversion. The cleaned up my_string is then passed into the _parse()_ function which attempts to convert it into a floating-point number.
>这段代码接收 my_string 参数，它将用户输入的内容保存下来，并提供给 _trim()_ 函数。_trim()_ 函数会删除掉字符串两侧可能存在的额外空行或空格。我们之所以需要 _trim()_ 是因为 _read_line()_ 函数在输入中附加了一个额外的空行，这会导致转换会出现异常。然后将清理了空格字符的 my_string 传递到 _parse()_ 函数中，该函数会尝试将其转换为浮点数。

If _parse()_ successfully converts my_string to a number, it returns Ok. Within that Ok, we can find our floating-point number. If the user entered something other than a number, then _parse()_ will not be able to complete the conversion. If _parse()_ can’t do its job, it returns Err.
>如果 _parse()_ 成功地将 my_string 转换为数字，则返回 Ok。在这个情况下，我们可以得到浮点数。如果用户输入的不是数字，那么 _parse()_ 将无法正常完成转换，它会返回 Err。

Within the curly brackets (the body) of our match expression, we tell the computer what to do based on what type _parse()_ returns:
>在匹配表达式的花括号（主体）中，我们根据 _parse()_ 返回的类型告诉计算机怎么做：

```rust
Ok(_s) => break _s,
Err(_err) => println!(“Try again. Enter a number.”)
```

**If the Result is Ok**, _parse()_ was able to convert the type. In that case, we call a break, which stops the loop from looping, and we return the value stored in the Ok, which has been placed in the _s variable.
>**如果结果是 Ok**，则表示 _parse()_ 能够转换该类型。这时，我们调用一个 break，停止循环，并返回存储在 Ok 中的值，这个值会被放在 _s 变量中。

**If the Result is Err**, _parse()_ was not able to convert the type. In that case, we tell the user to “Try again. Enter a number.” Since we do not call a break, the loop starts over.
>**如果结果是 Err**，_parse()_ 无法完成转换。这时，我们会告诉用户“重试一次。输入一个数字”。由于我们不调用 break，所以循环重新开始。

>If I had to explain the whole Result thing in one sentence, it would be: If a function returns a Result, a match expression can execute different code based on whether the Result is Ok or Err.

>如果必须用一句话解释 Result，那就是：如果一个函数返回 Result，一个匹配表达式可以根据结果是 Ok 还是 Err 来执行不同的代码。

## Using Result in Your Own Functions
>在你的函数中应用 Result

Now that you understand how to handle Result, you might want to use it in some of the functions you create.
>既然你已经了解了处理 Result 的方法，那么你可能会希望在你自己创建的函数中使用它。

Let’s look at an example.
>我们先看一个例子。

```rust
fn main(){
    let my_num = 50;
    
    fn is_it_fifty(num: u32) -> Result<u32, &’static str> {
        let error = “It didn’t work”;
        if num == 50 {
            Ok(num)
        } else {
            Err(error)
        }
    }
    match is_it_fifty(my_num) {
        Ok(_v) => println!(“Good! my_num is 50”),
        Err(_e) => println!(“Error. my_num is {:?}”, my_num)
    }
}
```

This program checks the value of _my_num_. If the value is 50, then it shows success, but if the value is not 50, it shows an error.
>这个程序检查 _my_num_ 的值。如果值为 50，则表示成功；如果不是，则表示错误。

The _is_it_fifty()_ function is the main specimen here. It is the declared function that returns a Result. Let’s look at in line by line.
>这段代码的主体是 _is_it_fifty()_ 函数。它是有返回结果的声明式函数。我们逐行看其中的代码。

First, we declare _my_num_ and set its value. Then, we declare the _is_it_fifty()_ function:
>首先，我们声明 _my_num_ 并给它赋值。然后，我们声明 _is_it_fifty()_ 函数：

```rust
fn is_it_fifty(num: u32) -> Result<u32, &’static str> {
```

In our declaration, we specify that the function will take a parameter named num that is a 32 bit unsigned integer type (u32). Next, we specify the function’s return type. It will return a Result. The type returned in the Result will be either u32 or a string literal (&’static str).
>在我们的声明中，我们指定该函数接收一个名为 num 的参数，其类型是 32 位无符号整数类型（u32）。接下来，我们指定函数的返回值类型。表示函数会返回一个结果，类型是 u32 或字串（&'static str）

Then, we enter the body of our _is_it_fifty()_ function.
>然后，我们编写 _is_it_fifty()_ 函数的函数体。

```rust
let error = “It didn’t work”;
if num == 50 {
    Ok(num)
} else {
    Err(error)
}
```

The code in the body is an if else expression. It evaluates the argument passed into the function.
>函数体中的代码是一个 if else 表达式。它用于判断传入的参数。

If it is 50, then the function will return the Result Ok. The Ok will contain the value that was passed into the function (_num_).
>如果值是 50，那么函数将返回 Ok 的 Result。Ok 中将会包含传递给函数的值（_num_）。

If the argument is anything other than 50, the function will return Result Err. The _Err_ will contain the value of the error variable which is, “It didn’t work.”
>如果参数不是 50，函数将返回 Err 的 Result。_Err_ 会包含错误变量的值，也即 “It didn’t work.”

Any time that function is used, the Result it returns must be handled. In our program, as with most Rust programs, that is done with a match expression. I described the match expression part earlier.
>无论何时使用该函数，都必须处理它返回的 Result。在我们的程序中，与大多数 Rust 程序一样，是通过一个匹配表达式完成的。我在之前已经描述过匹配表达式的部分。

The Result can also be handled with _unwrap()_ or _expect()_ — also explained earlier.
>Result 类型可以使用 _unwrap()_ 或 _expect()_ 来处理 —— 前面也已经解释过。

![](https://miro.medium.com/max/480/1*ZLronSWbmj4IwGoWepecHQ.gif)

## Summing It All Up
>总结

Result is a type returned by a function that indicates whether the function ran successfully.
>Result 是一个函数的返回类型，它表示函数是否成功运行。

Many of Rust’s built-in functions return a Result, and when that is the case, there is no way to avoid it. If a function returns a Result, it must be handled.
>Rust 的许多内置函数都是返回 Result 类型，如果是这样的话，就没有办法避开它。如果一个函数返回 Result，它必须要被妥善处理。

Common ways of handling a Result are the _unwrap()_ function, the _expect()_ function, and a match expression.
>处理 Result 常用的方法是使用 _unwrap()_ 和 _expect() 函数以及匹配表达式。

It is possible to return a Result from your own functions. It is a great way of accounting for errors.
>可以从自己定义的函数中返回 Result。这是处理错误的好办法。

That’s pretty much all you need to know about Rust’s Result type, but in case you want to learn a bit more, or understand where I gathered my information, below is a list of the resources I used.
>关于 Rust 的 Result 类型，你需要知道的就这些了，但是如果想了解更多信息，或者想知道我从哪儿收集的这些信息，可以参考下方的资源列表。

## Resources
>资源

* https://doc.rust-lang.org/std/result/
* https://doc.rust-lang.org/1.2.0/book/match.html
    * See section: matching on enums
* https://doc.rust-lang.org/1.30.0/book/first-edition/error-handling.html
* https://doc.rust-lang.org/rust-by-example/flow_control/match.html
* https://blog.jonstodle.com/things-i-enjoy-in-rust-error-handling/
* https://stevedonovan.github.io/rust-gentle-intro/6-error-handling.html
* https://doc.rust-lang.org/book/ch03-03-how-functions-work.html
* https://doc.rust-lang.org/std/result/enum.Result.html#method.expect
