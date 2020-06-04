```java
Flux.just("a", "b", "c")
                .filter("b"::equals)
                .map(s -> s.concat(" equals"))
                .subscribe(System.out::println);
```

Flux.just("a", "b", "c") 生成 Flux