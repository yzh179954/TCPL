# Section 2.7. 顶层定义

在let和lambda表达式中绑定的变量在表达式外部是不可见的，如果我们需要一个对象，它可能是一个函数，我们要让他除了被遮盖之外像let,fn,+一样随处可见，我们就需要另一种形式的绑定，顶层绑定。

```clojure
(def double-any
  (fn [f x] (f x x)))
```

在上面这段代码中，变量double-any通过def语句绑定上了一个函数。通过def绑定的变量可以在任何地方被访问，除非它被其他绑定遮盖住。

```clojure
(double-any + 1)  ;=> 2
(double-any list 1)  ;=> (1 1)
```

用来实现顶层绑定的def语句除了函数之外，也可以绑定任何对象。

```clojure
(def a 1)  ;=> user/a

a  ;=> 1
```

上面的例子将1和a绑定，在未被遮盖的条件下，变量a始终存在，且值为1。但是通常，你应该把顶层绑定用在一个函数上。

```clojure
(def xyz '(x y z))
(let [xyz '(1 2 3)] xyz)  ;=> (1 2 3)
```
上面这段代码演示了作用域的遮盖，当发生遮盖情形时，def绑定的变量和let,lambda绑定的变量并没由区别。

而我们在之前的章节中说过，函数的定义有两种形式，一种讲理，一种方便。我们说过了讲理的那种，现在看看方便的。

```clojure
(fn [x y] (+ x y))
#(+ %1 %2)  ;=> 等价于第一个表达式
```

示例中第二个表达式就是所谓方便的定义方法，%1，%2分别代表两个参数，将函数体写在#()中，就得到了和第一个表达式一样的运算结果。不要问我为什么，唯一的原因大概就是方便，只是当你参用第二种方式之后，需要注意两点，我们失去了复杂的参数匹配，当我们的函数体有多条语句的时候需要使用do表达式。do表达式的作用就是依次执行作为参数被传入的语句。

让我们来看更多例子
```clojure
(fn [x] (* x x))
#(* % %)

(fn [x y z] (list x y z))
#(list %1 %2 %3)

(fn [x y] (* x y) (+ x y) (- x y))
#(do (* %1 %2) (+ %1 %2) (- %1 %2))
```

对于第一个例子，当你的函数只接收一个参数的时候，只需要使用%代表参数就可以了。而第二个例子中，%1..%3分别代表第一二三个参数，如果你需要更多参数也是一样，把%后面跟的数字逐次加1就可以了。而最后一个例子，这段代码其实是没有意义的，但只是为了演示 do。它会依次计算x*y,x+y最后返回x-y的值，就和对应的lambda表达式所要做的事情一样。

对于顶层绑定，函数有一个自己专属的defn语句。它的用法和def一样，只不过糅合了lambda的味道。
```clojure
(def f (fn [x] (+ x x)))
(def f #(+ % %))
```
与上面两个表达式对应的defn表达式为
```clojure
(defn f [x] (+ x x))
```

它的通用形式为 (defn name [arg..argn] body1 .. bodyn)。除了第一位从fn变成的defn，[]前多了一个函数名之外，和lambda表达式没有区别。最大的不同就在于defn是顶层绑定，它所绑定的函数只要不被遮盖便可以在任何地方使用。

```clojure
(defn doubler [f] #(f % %))
```

我们用defn定义了一个全局可用的doubler函数，它接收一个参数f，返回了一个接收一个参数并传入f调用的函数。那么doubler函数用什么意义呢？
```clojure
(def dou (doubler +))
(dou 2)  ;=> 4
(def dou-lst (doubler list))
(dou-lst 1)  ;=> (1 1)
```

你有没有看迷糊呢？让我们从第一个表达式开始说起，(double +)返回的值是#(+ % %)的值，也就是(fn [x] (+ x x))的值，所以，def将dou和将参数相加的函数绑定，所以在第二个表达式中dou在第一位先求得#<dou>，然后调用#<dou>,也就是(fn [x] (+ x x))所以得到了4。而第三第四个表达式也是同理，不再赘述。

如果，我们往doubler中传入的参数不是函数会发生什么？很简单，一个执行态的表的第一位不是函数会发生什么？如果你已经忘了，打开repl随便试试吧。

### Exercise 2.7.1
如果将f绑定到#(+ % %)上，当你输入(f f f)时会发生什么？
