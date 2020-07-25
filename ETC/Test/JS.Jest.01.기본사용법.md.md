# 01. Jest 기본 사용법

> 

[🏠Home](https://github.com/batboy118/Study_Note)

[◀Previous page ](./README.md)

---

<!-- TOC -->

- - 

<!-- /TOC -->

## 1. TDD? (Test Driven Development)

- TDD : 테스트 코드를 먼저 짜고, 해당 함수를 이어서 짜는 방법

- 필요 환경
  - node.js
  - jest
    - `npm i -g jest`
- 유닛 테스트 : 함수 단위로 테스트하는 코드

## 2. jest 시작

- jestjs.io 에서 공식문서 확인 가능
- jest를 사용하기 위해서는 `pakage.json` 또는 `jest.config.js` 파일이 필요하다.

- 테스트할 폴더를 만들고 그 안에서 `npm init -y`명령어로 pakage.json 파일을 만든다.
-  `jest.test.js` 파일을 만들어 준다. (파일명은 프레임워크의 규칙이기 때문에 지켜주어야 한다.)

```js
//jest.test.js
test('init', () => {
	
});
```

위 코드를 적고, `jest.test.js`를 실행시키면 테스트가 동작된다.

위 코드를 아래와 같이 리팩터링 해보자.

```js
test('init', () => {
	let a = 1;
	expect(double(a)).toBe(a*2);
});

function double(a) {
	return a*2;
}
```

`jest jest.test.js`를 실행시키면 테스트가 통과된다.

> jest --watch 옵션
>
> 이 옵션을 사용하면 연속으로 테스트를 할 수 있다.
>
> `jest --watch jest.test.js`

- `expect(함수(입력값)).toBe(기대값)`는 같은 객체인지 확인할 때 쓰인다.
- `expect(함수(입력값)).toEqual(기대값)`는 같은 값을 가진 객체인지(즉, 실체는 다르지만 값이 같은지 확인 할 때) 확인할 때 쓰인다.
- `assert(함수(입력값), 기대값)`

## 3. jest 실습

`data.json 파일`

```json
[
    {
        "id": 1,
        "name": "Chicago Pizza",
        "image": "/images/chicago-pizza.jpg",
        "desc": "The pan in which it is baked gives the pizza its characteristically high edge which provides ample space for large amounts of cheese and a chunky tomato sauce.",
        "price": 9
    },
    {
        "id": 2,
        "name": "Neapolitan Pizza",
        "image": "/images/neapolitan-pizza.jpg",
        "desc": "This style of pizza is prepared with simple and fresh ingredients: a basic dough, raw tomatoes, fresh mozzarella cheese, fresh basil, and olive oil. A fantastic original pizza.",
        "price": 7
    },
    {
        "id": 3,
        "name": "New York Pizza",
        "image": "/images/ny-pizza.jpg",
        "desc": "New York-style pizza has slices that are large and wide with a thin crust that is foldable yet crispy. It is traditionally topped with tomato sauce and mozzarella cheese.",
        "price": 8
    },
    {
        "id": 4,
        "name": "Sicilian Pizza",
        "image": "/images/sicilian-pizza.jpg",
        "desc": "Sicilian pizza is pizza prepared in a manner that originated in Sicily, Italy. Sicilian pizza is also known as sfincione or focaccia with toppings. A great tasteful pizza all around.",
        "price": 9
    }
]
```

테스트 코드

```js
import pizzas from '../data';

// very basic test to notify the user if our pizza data has changed
test('the pizza data is correct', () => {
    expect(pizzas).toMatchSnapshot();
    expect(pizzas).toHaveLength(4);
    expect(pizzas.map(pizza => pizza.name)).toEqual([
        'Chicago Pizza',
        'Neapolitan Pizza',
        'New York Pizza',
        'Sicilian Pizza',
    ]);
});

// let's test that each item in the pizza data has the correct properties
for (let i = 0; i < pizzas.length; i += 1) {
    it(`pizza[${i}] should have properties (id, name, image, desc, price)`, () => {
        expect(pizzas[i]).toHaveProperty('id');
        expect(pizzas[i]).toHaveProperty('name');
        expect(pizzas[i]).toHaveProperty('image');
        expect(pizzas[i]).toHaveProperty('desc');
        expect(pizzas[i]).toHaveProperty('price');
    });
}

// default jest mock function
test('mock implementation of a basic function', () => {
    const mock = jest.fn(() => 'I am a mock function');

    expect(mock('Calling my mock function!')).toBe('I am a mock function');
    expect(mock).toHaveBeenCalledWith('Calling my mock function!');
});

// let's mock the return value and test calls
test('mock return value of a function one time', () => {
    const mock = jest.fn();

    // we can chain these!
    mock.mockReturnValueOnce('Hello').mockReturnValueOnce('there!');

    mock(); // first call 'Hello'
    mock(); // second call 'there!'

    expect(mock).toHaveBeenCalledTimes(2); // we know it's been called two times

    mock('Hello', 'there', 'Steve'); // call it with 3 different arguments
    expect(mock).toHaveBeenCalledWith('Hello', 'there', 'Steve');

    mock('Steve'); // called with 1 argument
    expect(mock).toHaveBeenLastCalledWith('Steve');
});

// let's mock the return value
// difference between mockReturnValue & mockImplementation
test('mock implementation of a function', () => {
    const mock = jest.fn().mockImplementation(() => 'United Kingdom');
    expect(mock('Location')).toBe('United Kingdom');
    expect(mock).toHaveBeenCalledWith('Location');
});

// spying on a single function of an imported module, we can spy on its usage
// by default the original function gets called, we can change this
test('spying using original implementation', () => {
    const pizza = {
        name: n => `Pizza name: ${n}`,
    };
    const spy = jest.spyOn(pizza, 'name');
    expect(pizza.name('Cheese')).toBe('Pizza name: Cheese');
    expect(spy).toHaveBeenCalledWith('Cheese');
});

// we can mock the implementation of a function from a module
test('spying using mockImplementation', () => {
    const pizza = {
        name: n => `Pizza name: ${n}`,
    };
    const spy = jest.spyOn(pizza, 'name');
    spy.mockImplementation(n => `Crazy pizza!`);

    expect(pizza.name('Cheese')).toBe('Crazy pizza!');
    spy.mockRestore(); // back to original implementation
    expect(pizza.name('Cheese')).toBe('Pizza name: Cheese');
});

// let's test pizza return output
test('pizza returns new york pizza last', () => {
    const pizza1 = pizzas[0];
    const pizza2 = pizzas[1];
    const pizza3 = pizzas[2];
    const pizza = jest.fn(currentPizza => currentPizza.name);

    pizza(pizza1); // chicago pizza
    pizza(pizza2); // neapolitan pizza
    pizza(pizza3); // new york pizza

    expect(pizza).toHaveLastReturnedWith('New York Pizza');
});

// let's match some data against our object
test('pizza data has new york pizza and matches as an object', () => {
    const newYorkPizza = {
        id: 3,
        name: 'New York Pizza',
        image: '/images/ny-pizza.jpg',
        desc:
        'New York-style pizza has slices that are large and wide with a thin crust that is foldable yet crispy. It is traditionally topped with tomato sauce and mozzarella cheese.',
        price: 8,
    };
    expect(pizzas[2]).toMatchObject(newYorkPizza);
});

// async example, always return a promise (can switch out resolves with reject)
test('expect a promise to resolve', async () => {
    const user = {
        getFullName: jest.fn(() => Promise.resolve('Karl Hadwen')),
    };
    await expect(user.getFullName('Karl Hadwen')).resolves.toBe('Karl Hadwen');
});

test('expect a promise to reject', async () => {
    const user = {
        getFullName: jest.fn(() =>
                             Promise.reject(new Error('Something went wrong'))
                            ),
    };
    await expect(user.getFullName('Karl Hadwen')).rejects.toThrow(
        'Something went wrong'
    );
});
```

