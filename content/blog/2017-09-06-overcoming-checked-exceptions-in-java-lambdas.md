---
title: "Overcoming Checked Exceptions in Java Lambdas"
image: "images/post/2017-09-06-overcoming-checked-exceptions-lambda.jpg"
date: "2017-09-06T19:57:00+01:00"
author: "Frank Neff"
tags: ["exceptions", "lambda", "java"]
categories: [ "Programming", "Functional Programming"]
draft: false
---

In Java 8, the long awaited Lambda came to live, making it easy(-er) to do FP in Java. One problem I came across is, 
that most Java code throws checked exceptions which leads to IMHO ugly try/catch blocks in lambdas:

<!--more-->

```java
Function<A, B> fun = (a: A) -> {
    try {
        // some function call that trows checked exception$
        return callFn(a);
    } catch (Exception e) {
        // return failure result
    }
};
```

## The Good, the Bad and the Ugly

A really simple, but also not really nice option is to wrap thrown exceptions into an unchecked one:

```java
@FunctionalInterface
public interface CheckedFunction<T, R> extends java.util.function.Function<T, R> {
    
    @Override
    default R apply(T t) {
        try{
            return applyChecked(t);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    
    R applyChecked(T t) throws Exception;
}
```

Now we have an ugly `Function<T, R>` which still has a side effect, but a hidden one, because the `RuntimeException` will
bubble without being checked. **This works, but is really ugly!**

## Preventing Observable Side Effects

To prevent the observable side effects (and yes, every thrown exception is one), one could create a return type containing the 
functions result or an exception instance:

```java
import java.util.function.Function;

@FunctionalInterface
public interface ResultFunction<T, R> extends Function<T, ResultFunction.Result<R>> {

    @Override
    default Result<R> apply(T t) {
        try{
            return Result.success(applyChecked(t));
        } catch (Exception e) {
            return Result.error(e);
        }
    }

    R applyChecked(T t) throws Exception;

    final class Result<T> {
        private final T result;
        private final Exception error;

        private Result(T result, Exception error) {
            this.result = result;
            this.error = error;
        }

        static <A> Result<A> success(A result) {
            return new Result<>(result, null);
        }

        static <A> Result<A> error(Exception error) {
            return new Result<>(null, error);
        }

        public boolean isSuccess() {
            return error == null;
        }

        public <A> Result<A> map(Function<T, A> f) {
            return isSuccess() ? Result.success(f.apply(result)) : Result.error(error);
        }

        public <A> A fold(Function<Exception ,A> errorFn, Function<T, A> successFn) {
            return isSuccess() ? successFn.apply(result) : errorFn.apply(error);
        }

        public T getOrElse(Function<Exception, T> f) {
            return isSuccess() ? result : f.apply(error);
        }
    }

}
```

The `Result<T>` class is a generic disjoint union (some would call it `Either<A,B>`) which can have a `result` or an 
`error`. The `fold` method is convenient to forther process the result, because it just takes two functions 
(one for the error-case and one to process the result) which must return the same type. The `ResultFunction<T, R>` can 
be used (and defined) like a normal `Function<T, R>` but will return a `Result<T>` instead of throwing exceptions.

This implementation can be tested using the following simple main-method:

```java
public class Main {

    private static ResultFunction<String, Integer> strPlusOne = in -> Integer.valueOf(in) + 1;

    public static void main(String... args) {
        System.out.println(res("42")); // Success: 43
        System.out.println(res("foo")); // Error: input was not a number!

        System.out.println(strPlusOne.apply("bar").
            map(Object::toString).
            getOrElse(e -> "NaN"));
    }

    private static String res(String in) {
        return strPlusOne.apply(in).fold(
                e -> "Error: input was not a number!",
                r -> "Success: " + r
        );
    }
}
```

As you can see in the `res` helper method, `fold` can be used to return either a success or an error string depending on 
the `Result` state. Or, linke in the second example, `map` and `getOrElse` can be used to achieve the same. This 
entirely depends on how you are used to code...

Now, a lambda throwing exceptions (`NumberFormatException` in that case), can be used without any 
side effect. Of course similar implementations do also work for other functional interfaces like `BiFUnction`, 
`Supplier`, etc.

## Edit: Use FunctionalJava if Possible

An even better approach to handle exceptions in lambdas is, if you're able to, use the Try-function of the
[FunctionalJava library](http://www.functionaljava.org/). It does pretty much the same, but is more sophisticated and you
don't have to write it on your own. Give it a try and use other functional sugar like partial functions, products, 
unions, immutable collections, and so on...