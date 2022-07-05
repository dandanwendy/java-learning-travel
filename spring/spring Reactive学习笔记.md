[toc]

反应式流规范可以总结为4个接口：Publisher、Subscriber、 Subscription和Processor。Publisher负责生成数据，并将数据发送给 Subscription（每个Subscriber对应一个Subscription）。

Publisher接口声明 了一个方法 subscribe()，Subscriber可以通过该方法向 Publisher发起订阅。

至于Processor接口，它继承自Subscriber和Publisher，当作为 Subscriber时，Processor会接收数据并以某种方式对数据进 行处理。然后它会将角色转变为Publisher，并将处理的结果发布给它的 Subscriber。 



# springboot集成reactive

## maven依赖

```xml
<!--核心依赖-->
<dependency> 
  <groupId>io.projectreactor</groupId> 
  <artifactId>reactor-core</artifactId> 
</dependency>
<!--测试依赖-->
<dependency> 
    <groupId>io.projectreactor</groupId> 
    <artifactId>reactor-test</artifactId> 
    <scope>test</scope> 
</dependency>
```

## Mono和Flux

Mono和Flux是Reactor的两种核心类型之一，两者都实现了反应式流的Publisher接口。

```java
    public static void main(String[] args) {
        Mono.just("Craig")
                .map(n -> n.toUpperCase())
                .map(cn -> "Hello, " + cn + "!")
                .subscribe(System.out::println);
    }
```



# reactive基本方法

## 创建操作

在Spring中使用反应式类型时，我们通常将会从repository或者 service中获取Flux或Mono，并不需要我们自行创建。即通常我们使用dao层或service层返回的Flux和Mono对象。

偶尔，我们可能需 要创建一个新的反应式Publisher。 Reactor提供了多种创建Flux和Mono的操作。这里，我们将介绍其 中一些有用的创建操作。 

### 对象创建

**just()**

如果你有一个或多个对象，并想据此创建Flux或Mono，那么可以 使用Flux或Mono上的静态 just()方法来创建一个反应式类型。

```java
        // 生产者
        Flux<String> fruitFlux = Flux .just("Apple", "Orange", "Grape", "Banana", "Strawberry");
```

此时我们已经创建了Flux，但是它还没有订阅者。如果没有任何的 订阅者，那么数据将不会流动。要添加一个订阅者，我们可以在Flux上调用subscribe()方法，传递给 subscribe()方法的lambda表达式实际上是一个 java.util.Consumer，用来创建反应式流的Subscriber。在调用subscribe() 之后，数据会开始流动。

```java
        // 订阅者
        fruitFlux.subscribe( f -> System.out.println("Here's some fruit: " + f) );
```

测试Flux或Mono更好的方法是使用Reactor提供的 StepVerifier，而不是打印到控制台。对于给定的Flux或Mono，StepVerifier将会订阅该反应式类 型，在数据流过时对数据应用断言，并在最后验证反应式流是否按预期完成。

```java
        StepVerifier.create(fruitFlux)
                .expectNext("Apple")
                .expectNext("Orange")
                .expectNext("Grape")
                .expectNext("Banana")
                .expectNext("Strawberry")
                .verifyComplete();
```

### 数组创建

要根据数组创建Flux，可以调用Flux上的静态方法fromArray()，并传入一个源数组

```java
    static private void createFromArray() {
        String[] fruits = new String[]{
                "Apple", "Orange", "Grape", "Banana", "Strawberry"};
        Flux<String> fruitFlux = Flux.fromArray(fruits);

        StepVerifier.create(fruitFlux)
                .expectNext("Apple")
                .expectNext("Orange")
                .expectNext("Grape")
                .expectNext("Banana")
                .expectNext("Strawberry")
                .verifyComplete();
    }
```



### 集合创建

如果我们需要根据java.util.List、java.util.Set或者其他任意 java.lang.Iterable 的实现来创建Flux，那么可以将其传递给静态的fromIterable()方法

```java
    static private void createFromList() {
        List<String> fruitList = new ArrayList<>();
        fruitList.add("Apple");
        fruitList.add("Orange");
        fruitList.add("Grape");
        fruitList.add("Banana");
        fruitList.add("Strawberry");
        Flux<String> fruitFlux = Flux.fromIterable(fruitList);
        // ... verify steps
    }
```

### stream创建

```java
    static private void createFromStream() {
        Stream<String> fruitStream = Stream.of("Apple", "Orange", "Grape", "Banana", "Strawberry");
        Flux<String> fruitFlux = Flux.fromStream(fruitStream);

        // ... verify steps
        // 订阅者
        fruitFlux.subscribe(f -> System.out.println("Here's some fruit: " + f));
    }
```

### 自动生成

有时候我们根本没有可用的数据，而只是想要一个作为计数器的 Flux，它会在每次发送新值时增加1。

**range()**

要创建一个计数器Flux，我们可以使用静态方法range()。创建一个区间Flux，起始值为1，结束值为 5

```java
Flux<Integer> intervalFlux = Flux.range(1, 5);
```

**interval()**

另一个与range()方法类似的Flux创建方法是interval()。interval()的特殊 之处在于，我们不是给它设置一个起始值和结束值，而是指定一个应该每隔多长时间发出值的间隔时间。

```java
Flux<Long> intervalFlux2 = Flux.interval(Duration.ofSeconds(1)) .take(5);
```

通过interval()方法创建的Flux会从0开始发布值，并且后续的条目依次递增。此外，因为interval()方法没有指定最大值，所以它可能会永远运行。我们也可以使用take()方法将结果限制为前5个条目。

## 组合操作

有时候，我们会需要操作两种反应式类型，并以某种方式将它们合并在一起。或者，在其他情况下，我们可能需要将Flux拆分为多种反应 式类型。组合操作就是合并或拆分Flux的。

### 合并

**mergeWith()**

```java
    // 合并两个Flux
    public void mergeFluxes() {
        Flux<String> characterFlux = Flux
                .just("Garfield", "Kojak", "Barbossa")
                .delayElements(Duration.ofMillis(500));

        Flux<String> foodFlux = Flux.
                just("Lasagna", "Lollipops", "Apples")
                .delaySubscription(Duration.ofMillis(250))
                .delayElements(Duration.ofMillis(500));

        Flux<String> mergedFlux = characterFlux.mergeWith(foodFlux);
        StepVerifier.create(mergedFlux)
                .expectNext("Garfield")
                .expectNext("Lasagna")
                .expectNext("Kojak")
                .expectNext("Lollipops")
                .expectNext("Barbossa")
                .expectNext("Apples")
                .verifyComplete();
    }
```

通常，Flux 会尽可能快地发布数据。因此，我们在创建的两个Flux流上使用delayElements()方法来减慢它们的速度——每500毫秒发布一个条目。此外，为了使食物Flux在角色名称Flux之后再开始流式传输，我们调用了食物Flux上的delaySubscription()方法，以便它在订阅后再经过 250毫秒后才开始发布数据。

在合并后的 Flux中会交错在一起，结果是：一个角色、一个食物、另一个角色、另一个食物，以此类推。

## 转换操作

### 过滤方法

在数据流经一个流时，我们通常需要过滤掉某些值并对其他的值进 行处理。

**skip()**

skip操作将创建一个新的Flux，它会首先跳过指定数量的数据项，然后从源 Flux 中发布剩余的数据项。

```java
    public void skipAFew() {
        Flux<String> skipFlux = Flux
                .just("one", "two", "skip a few", "ninety nine", "one hundred")
                .skip(3);
        StepVerifier.create(skipFlux)
                .expectNext("ninety nine", "one hundred")
                .verifyComplete();
    }
```



**take()**

skip操作会跳过前面几个数据项，而take操作只发布第一批指定数量的数据项，然后将取消订阅.

```java
    public void take() {
        Flux<String> nationalParkFlux = Flux
                .just("Yellowstone", "Yosemite", "Grand Canyon", "Zion", "Grand Teton")
                .take(3);
        StepVerifier.create(nationalParkFlux)
                .expectNext("Yellowstone", "Yosemite", "Grand Canyon")
                .verifyComplete();
    }
```

与skip()方法一样，take()方法也有另一种替代形式，基于间隔时间而不是数据项个数。此处不细表。

**filter()**

skip操作和take操作都可以被认为是过滤操作，其过滤条件是基于计数或者持续时间的，而Flux值的更通用过滤则是filter操作。其实只需掌握filter()即可。过滤条件我们可以自己实现，很灵活也很方便。

我们需要指定一个Predicate，用于决定数据项是否能通过Flux，filter操作允许我们根据任何条件进行选择性地发布。

```java
    public void filter() {
        Flux<String> nationalParkFlux = Flux
                .just("Yellowstone", "Yosemite", "Grand Canyon", "Zion", "Grand Teton")
                .filter(np -> !np.contains(" "));
        
        StepVerifier.create(nationalParkFlux)
                .expectNext("Yellowstone", "Yosemite", "Zion")
                .verifyComplete();
    }
```

**distinct()**

想要过滤掉已经接收过的数据条目，即给数据去重，可以采用distinct操 作

```java
    public void distinct() {
        Flux<String> animalFlux = Flux
                .just("dog", "cat", "bird", "dog", "bird", "anteater")
                .distinct();
        
        StepVerifier.create(animalFlux)
                .expectNext("dog", "cat", "bird", "anteater")
                .verifyComplete();
    }
```



### 映射方法

## 逻辑操作

# reactive web层

# reactive持久层