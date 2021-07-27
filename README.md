```jsx
let data = { price: 5, quantity: 2 };
let target = null;

// Our simple Dep class
class Dep {
  constructor() {
    this.subscribers = [];
  }
  depend() {
    if (target && !this.subscribers.includes(target)) {
      // Only if there is a target & it's not already subscribed
      this.subscribers.push(target);
    }
  }
  notify() {
    this.subscribers.forEach(sub => sub());
  }
}

// Go through each of our data properties
Object.keys(data).forEach(key => {
  let internalValue = data[key];

  // Each property gets a dependency instance
  const dep = new Dep();

  Object.defineProperty(data, key, {
    get() {
      dep.depend(); // <-- Remember the target we're running
      return internalValue;
    },
    set(newVal) {
      internalValue = newVal;
      dep.notify(); // <-- Re-run stored functions
    }
  });
});

// The code to watch to listen for reactive properties
function watcher(myFunc) {
  target = myFunc;
  target();
  target = null;
}

watcher(() => {
  data.total = data.price * data.quantity;
});

console.log("total = " + data.total)
data.price = 20
console.log("total = " + data.total)
data.quantity = 10
console.log("total = " + data.total)
```

```jsx
total = 10
total = 40
total = 200
```

전체적으로 목표로 하는 것은
- 어떤 대상 객체에 대해서 (`data`)
- 어떤 속성이 변화할 때 어떤 처리를 이어서 하게 하고 싶은 것 (reactive 한 속성인 `data.price`, `data.quantity` 그리고 처리인 `() => {data.total = data.price * data.quantity;}` )

이것이 이뤄지게 하려면, 대상 객체의 각 속성에 대해 `defineProperty` 를 이용해서 
- `set` 과정에서 값을 mutating 하고 종속된 처리 (`() ⇒ {data.total = ...}`) 를 실행한다.
- `get` 과정에서 종속 시키고자 하는 처리가 있다면 담은 뒤 (`dep`) 해당 값을 (`internalValue`) 반환한다.
    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6108168e-8bb1-4093-abe3-2d478b147d5d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6108168e-8bb1-4093-abe3-2d478b147d5d/Untitled.png)

이런 과정에서 `dep` 에 처리할 것을 올리는 것이 중요해지는데, 
```jsx
let **target** = null;

// ...

function watcher(myFunc) {
  **target** = myFunc;
  **target**();
  **target** = null;
}

watcher(**() => {
  data.total = data.price * data.quantity;
}**);

// ...

    get() {
      dep.**depend**(); // <-- Remember the target we're running
      return internalValue;
    },

// ... Class Dep

  **depend**() {
    if (**target** && !this.subscribers.includes(target)) {
      // Only if there is a target & it's not already subscribed
      this.**subscribers.push**(target);
    }
  }
```

위와 같이 Dep subscribers 에 reactive 하게 처리해야 할 것들을 담아두고, 그것을 reactive property 를 `set` 할 때 마다, `.notify` 를 통해서 호출하게 된다.
```jsx
// ...

    set(newVal) {
      internalValue = newVal;
      dep.notify(); // <-- Re-run stored functions
    }

// ... Class Dep

  notify() {
    this.subscribers.forEach(sub => sub());
  }
```
