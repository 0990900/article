# Validation for TypeScript

## History 1 - 값의 검증

자바스크립트의 검증은 대게 유저에게 받은 이벤트로 폼을 검증하는 것으로 시작되었다.

> [validator.js](https://github.com/validatorjs/validator.js)는 값을 검증하는 가장 유명한 도구

``` js
var validator = require('validator');

const email = event.text;
validator.isEmail(email);

```

## History 2 - 객체의 검증 (1)

값의 검증만으로는 기능이 부족하므로 [JSON Schema](https://json-schema.org/)를 도입하기 시작.

스키마를 통해 인스턴스를 체계적으로 검증하는 방법으로 진화한다.

> JSON Schema 또한 JSON이다. 


## History 3 - 객체의 검증 (2)

타입스크립트가 나오면서 자바스크립트에 타입의 개념을 넣을 수 있게 되었고, 타입스크립트의 데코레이터를 이용하여 검증을 하고자 하는 시도가 생김.

> [class-validator](https://github.com/typestack/class-validator)

``` ts
export class Post {
  @Length(10, 20)
  title: string;

  @Contains('hello')
  text: string;

  @IsInt()
  @Min(0)
  @Max(10)
  rating: number;

  @IsEmail()
  email: string;

  @IsFQDN()
  site: string;

  @IsDate()
  createDate: Date;
}
```

`JSON Schema`보다 직관적인 설정이기 때문에 이해하기도 쉽고 표현하기도 쉬운 장점이 있다.

## History 4 - 함수 합성을 사용하여 재사용성 극대화

함수형 프로그래밍의 영향으로 코덱이라는 개념을 첨가한다.

> 코덱은 타입의 디코딩과 인코딩의 역할을 수행한다.

> Codec<A, O, I>이 있을 때, I => A 를 디코딩(Read)이라고 하고, A => O 를 인코딩(Write)이라고 한다.

디코딩과 인코딩에 사용되는 기능을 함수의 합성으로 만들 수 있다.
