# Free Monad

## 첫 번째 Free Monad - 거북이 다루기

거북이가 한 마리 있다.

그리고 다음의 명령을 이해할 수 있다고 가정한다.

- 움직이고 멈출 수 있다.
- 방향도 바꿀 수 있다.

아래와 같은 명령을 내린다면 어떻게 구현을 할 것인가?

- 100m 이동
- 45도 방향을 바꾼다
- 50m 이동
- 멈춘다


### 단순하게 접근해보기

거북이의 기능을 일반 함수로 표현해보자.

``` typescript
const move = (meter: number) => {};
const stop = () => {};
const turn = (degree: number) => {};
```

위의 문제는 아래처럼 표현할 수 있다.

``` typescript
const program = () => {
	move(100); // 100m 이동
	turn(45);  // 45도 방향을 바꾼다
	move(50);  // 50m 이동	
	stop();    // 멈춘다
}

// 이 함수의 의미는 나중에 나온다.
const compiler = (p: () => void) => p();
```

우리는 일반적인 함수로 간단하게 해결했다.

### 좀 더 생각해보기

좀 전에 만든 함수는 매우 직관적이고 심플하다.

이대로도 잘 동작하지만 제대로 된 함수형 프로그래밍이라고 할 수 없다.

왜냐하면, `program`의 입장에서 move, turn, stop 의 함수는 `program`과 전혀 상관없는 `Side Effect` 이기 때문이다.
> `program`과 move, turn, stop 함수는 서로 어떠한 연관도 없다.

또한 너무 심플한 나머지 요구사항이 추가되면 많은 수정이 필요하다.
수정 작업이 계속 `program`내의 `Side Effect`를 유발한다.


### 좀 더 함수형 프로그래밍으로 변경하기

이런 방법은 어떤가? 
> 명령을 정의 할 때, 다음 명령을 함께 전달하는 구조

마치 `Linked List`처럼 함수의 정의가 이어진다.
> const program = f -> f -> f -> ...

그러기 위해서는 어떤 명령어의 조합이라도 동일한 타입으로 강제해야 한다.
> program = move(turn)
> 
> program = stop(move)
> 
> program = turn(move, stop())

그래서 아래처럼 타입을 미리 정의하고 시작한다.

``` typescript
type Command = Move | Turn | Stop
```

실제는 이렇게 구현이 가능할 것이다.

``` typescript
// Case 1
const program = move(50, turn(45, stop()));

// Case 2
const program = stop(turn(45, move(50)));

// (참고) 순서는 다음과 같다고 가정한다.
// move(50) -> turn(45) -> stop()
```

하지만 구현에 따라 2가지 케이스가 생기게 된다.
둘 다 장단점이 분명하므로 하나만 선택해야 한다.

`Case 1`
- 순방향으로 명령어가 중첩이 되니까 순차적으로 실행하면 되는 이점
	- move -> turn -> stop
- 구성 후 마지막에 명령어 추가 불가
	- stop(program) 이라고 정의하면, 명령어가 **맨 앞**에 추가된다.

`Case 2` 
- 역방향으로 명령어가 중첩이 되니까 순차적으로 실행하려면 모든 명령어를 뒤집은 뒤에 실행
	- move <- turn <- stop
- 구성 후 마지막에 명령어 추가 가능
	- stop(program) 이라고 정의하면, 명령어가 **맨 뒤**에 추가된다.

둘다 참조투명성을 보장한다.

또한 `program`과 move, turn, stop은 같은 타입이라는 관계에 놓여있기 때문에 어떤 순서라도 조합이 가능하다.
> move, turn, stop이 더 이상 `Side Effect`가 아니다.

그렇다면 지금부터는 선택의 영역이다.
> (Case 1) 빠른 실행을 할 것인가 VS (Case 2) 마지막에 계속 명령을 추가할 것인가

나는 매번 실행할 때마다 뒤집어 실행하는 것이 마음에 들지 않으므로 `Case 1`을 선택했다.

### 구현하기

순방향으로 구현을 해보면, 다음과 같다.

``` typescript
type Move = (path: string[]) => void;
type Turn = (path: string[]) => void;
type Stop = (path: string[]) => void;
type Command = Move | Turn | Stop;

const move = (meter: number, next?: Command): Move => (path: string[]) => {
    path.push(`Move ${meter} meters`);
    next && next(path);
}

const turn = (degree: number, next?: Command): Turn => (path: string[]) => {
    path.push(`Turn ${degree} degrees`);
    next && next(path);
}

const stop = (next?: Command): Stop => (path: string[]) => {
    path.push(`Stop`);
    next && next(path);
}

const program: Command = move(50, turn(45, stop()));
const compiler = (program: Command): Array<string> => {
		const path = [] as Array<string>;
		program(path);
		return path;
}
```


### 결론

정의된 대로 위의 문제를 풀면,

``` typescript
console.log(compiler(move(100, turn(45, move(50, stop())))));
```

결국 거북이를 다루는 프로그램은 `Move`, `Turn`, `Stop`의 언어로 이루어진다.

여기에 다른 명령어가 추가된다고 해도 `Command`의 서브 타입을 추가하는 방법으로 쉽게 해결이 가능하다.

> `Free Monad`를 표현하는 방법은 언어에 따라 모두 다르다. 하지만 기본 개념은 동일하다.

## Free Monad - 정의

[from haskell wiki](https://wiki.haskell.org/Free_monad)

> A free monad generated by a functor is a special case of the more general free (algebraic) structure over some underlying structure. For an explanation of the general case, culminating with an explanation of free monads, see the article on free structures.

문제를 해결하기 위해 적용하고 싶은 효과가 특정한 형식을 가지고 반복적이거나 재귀적인 성격을 띄는 경우 사용할 수 있다.

다른 `Monad` 처럼 어떠한 효과 적용하여 문제를 해결하기 보다는, 중첩된 `구조`를 형성하여 문제를 해결하는데 더 의미를 둔다.

> 구조가 `List`나 `Tree` 모양으로 구조가 형성된다고 이해하면 쉽다.

이러한 구조를 `Program`이라고 하고, `Program`을 해석하는 함수를 `Interpeter` 또는 `Compiler` 라고 한다.

즉, `Free Monad`란 어떤 문제를 `Free Monad`로 만들어진 구조로 표현하고 `Interpreter` 또는 `Compiler`로 해석하여 해결한다.

## 두 번째 Free Monad - Trampoline

### 모델링

[트램폴린](https://ko.wikipedia.org/wiki/트램펄린_(컴퓨팅))은 재귀함수가 스택을 소모하지 않고 동작하도록 변경하는 테크닉을 말한다.

이는 재귀함수를 구성할 때 Trampoline 기법을 사용하면 어떤 재귀함수든지 [공재귀:co-recursion](https://en.wikipedia.org/wiki/Corecursion) 스타일을 강제한다.

이는 언어가 Tail Recursion Optimization(TCO)를 지원하지 않은 경우 대체할 수 있다.

`Trampoline`을 설계할 때는 재귀함수의 상태를 2가지로 나눠서 모델링을 한다.

- `Suspend` 
	- 재귀함수 중간 상태. 
	- 이전 재귀함수의 결과값을 받아서 현재 재귀함수를 실행한다.
- `Done`
	- 재귀함수 종료


``` typescript
type Trampoline<A> = Suspend<A> | Done<A>;

type Suspend<A> = {f: (...args: any[]) => Trampoline<R>};
type Done<A> = {value: A};
```


`Trampoline`은 `Suspend`와 `Done`의 프로그램이라고 할 수 있다.

### 예시

우선 스택을 소모하는 재귀함수 스타일의 피보나치 프로그램을 보면 다음과 같다.

``` typescript
const fibonacci = (n: number): number => {
  const f = (a: number, b: number, count: number): number => {
    return count === n 
	    ? a + b 
	    : f(b, a + b, count + 1); // 스택을 소모
  };
  return f(0, 1, 0);
};
console.log(fibonacci(10000));
```

`Trampoline`을 이용한 피보나치 수열을 예로 들어보면, 다음과 같다.

``` typescript
const fibonacci = (n: number): Trampoline<number> => {
  const f = (a: number, b: number, count: number): Trampoline<number> => {
    return count === n
      ? done(a + b)
      : suspend(() => f(b, a + b, count + 1));
  };
  return f(0, 1, 0);
};
console.log(compiler(fibonacci)(10000));
```

테스트 코드는 [여기](https://github.com/0990900/try-typescript/blob/main/test/trampoline.test.ts)에 있다.

### 구현

컴파일러를 구현해보자.

1. 컴파일러는 `Trampoline`을 리턴하는 함수를 파라미터로 받는다.
2. 재귀적으로 호출하여 어떤 타입의 인스턴스인지 확인한다.
3. Suspend 타입인 경우 이전에 호출한 값을 그 다음 Suspend 함수에 전달한다.
4. Done 타입인 경우 리턴한다.

``` typescript
const compiler = <A>(f: (...args: any[]) => Trampoline<A>, context?: any) => {
    const tryBind = (supplier: (...args: any[]) => Trampoline<A>) => context ? supplier.bind(context) : supplier
    return (...args: any[]): A => {
        let result: Trampoline<any> = tryBind(f)(...args)
        while (true) {
            if (result instanceof Suspend) {
                result = tryBind(result.supplier)()
            } else if (result instanceof Done) {
                return result.a
            } else {
                throw new Error("Trampoline 인스턴스가 아님")
            }
        }
    }
};
```

초기에 구현된 컴파일러는 [여기](https://github.com/0990900/try-typescript/blob/main/src/trampoline-old.ts)에 있고, 좀 더 가다듬은 컴파일러는 [여기](https://github.com/0990900/try-typescript/blob/main/src/trampoline.ts)에 있다.

## 그 외에도

구조가 있는 객체를 데이터베이스에서 읽고 구성할 때도 사용할 수 있다.

데이터베이스에 질의한 쿼리에 대한 ResultSet에서 새로운 객체를 구성하는 것을 재귀적으로 구성한다면 가능하지 않을까?

참조 할 만한 다른 링크를 첨부한다.

- [Easy Scalaz 5 – (Co) Yoneda, Free Monad, and Trampoline](https://1ambda.blog/2018/06/30/easy-scalaz-5/)
- [다른 예제 - Kotlin, Scala](https://github.com/eunmin/free-monad)
- [Move Over Free Monads: Make Way for Free Applicatives](https://www.youtube.com/watch?v=H28QqxO7Ihc) 영상에 보면 `Free Functor Hierarchy`를 설명하면서 그중에 하나로 `Free Monad`를 표현하고 있다.
	- Free Functors - Programs that Change Values
	- Free Applicative - Programs that Build Data
	- **Free Monads** - Programs that Build Programs
