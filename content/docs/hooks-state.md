---
id: hooks-state
title: Using the State Hook
permalink: docs/hooks-state.html
next: hooks-effect.html
prev: hooks-overview.html
---

*Hooks* là một tính năng mới được thêm vào từ phiên bản 16.8. Cho phép sử dụng state và các tính năng React mà không cần viết class

Ở [trang giới thiệu](/docs/hooks-intro.html) có ví dụ này để làm quen với Hooks

```js{4-5}
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Chúng ta bắt đầu tìm hiểu Hooks bằng cách so sánh với equivalent class

## Ví dụ Equivalent Class {#equivalent-class-example}

Nếu trước đây bạn sử dụng class thì điều này khá quen thuộc

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

State bắt đầu là {count: 0}, và tăng `state.count` lên 1 khi click vào button bằng cách gọi `this.setState()`

## Hooks và Function Components {#hooks-and-function-components}

Chú ý, các thành phần chức năng trong React sẽ như thế này

```js
const Example = (props) => {
  // Sử dụng Hooks tại đây
  return <div />;
}
```

hoặc:

```js
function Example(props) {
  // Sử dụng Hooks tại đây
  return <div />;
}
```

Từ trước, chúng ta đã biết những thứ này gọi là "stateless components". Bây giờ chúng tôi giới thiệu khả năng sử dụng React state từ những trạng thái này, và chúng tôi thích cái tên "function components"

Hooks **không** hoạt động bên trong các class, nhưng bạn có thể sử dụng chúng thay vì viết các lớp.

## Hook là gì? {#whats-a-hook}

Trong ví dụ này, chúng tôi bắt đầu bằng cách nhập `useState` Hook tử React

```js{1}
import React, { useState } from 'react';

function Example() {
  // ...
}
```

**Hook là cái gì?** Hook là một function đặc biết cho phép bạn "hook into" các tính năng của React. Ví dụ `useState` là một Hook cho phép bạn thêm React state vào function components

**Khi nào thì tôi sử dụng Hook?** Nếu bạn viết một function component và nhận ra rằng bạn cần thêm một số state cho nó, trước đây bạn phải chuyển đổi nó về class. Nhưng bây giờ có thể sử dụng Hook bên trong function component đã tồn tại. Thực hiện thôi, Let's go!

>Chú ý:
>
>Có một số quy tắc đặc biệt mà bạn không thể sử dụng Hooks trong một component. Chúng ta sẽ học nó trong [quy tắc trong Hooks](/docs/hooks-rules.html).

## Khai báo State {#declaring-a-state-variable}

Trong class, khởi tạo `count` và gán cho nó bằng `0` bằng cách đặt `this.state` thành {count: 0} trong contructor:

```js{4-6}
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
```

Trong function component, chúng ta không có `this`, vì vậy chúng ta không thể thể gán hoặc đọc `this.state`. Thay vào đó, chúng ta gọi `useState` Hook trực tiếp bên trong component: 

```js{4,5}
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);
```

**Gọi `useState` để làm gì?** Nó khai báo "biến state". Biến có thể được đặt là `count` hoặc bất kì cái tên khác bạn muốn ví dụ `sơnTùng`. Đây là một cách để "preserve" một số giá trị khi gọi function - `useState` là một cách mới để sử dụng `this.state` quy ước trong class


**Làm cách nào để dùng `useState` như một đối số?** Đối số duy nhất cho `useState()` Hook là khi khởi tạo state. Không giống như class, state không phải là một object, chúng ta có thể tạo kiểu number hoặc string nếu chúng ta cần. Trong ví dụ, chúng tôi chỉ muốn hiển thị số lần người dùng click vào button, vì thế mặc định ban đầu là 0.(Nếu muốn lưu 2 giá trị khác nhau ở hai trạng thái, thì sẽ phải gọi `useState()` hai lần);

**`useState` trả về cái gì?** Nó trả về một cặp giá trị: state hiện tại và function thực hiện cập nhật nó. Đây là lý do tại sao chúng ta viết `const [count, setCount] = useState`. Điều này tương tự như `this.state.count` và `this.setState`.

Bây giờ chúng ta đã biết `useState` Hook làm gì. Vậy nhìn vào ví dụ dưới đây, bạn đã hiểu chưa nào?

```js{4,5}
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);
```

Chúng tôi khai báo một biến state đặt tên là `count`, và gán cho nó giá trị bằng `0`. React sẽ ghi nhớ giá trị hiện tại của nó giữa các lần render lại, và cung cấp biến gần nhất cho function. Nếu muốn cập nhật lại `count`, chúng tôi sẽ gọi `setCount`.

>Chú ý
>
>Bạn tự hỏi: Tại sao `useState` không được đặt tên là `createState`?
>
>"Create" có thể là khá chính xác vì state được tạo lần đầu tiên khi render. Trong lần render tiếp theo `useState` là chính xác hơn bởi nó cung cấp cho chúng ta giá trị `state` hiện tại. Nếu không, nó sẽ không phải là "state". Có một lý do mà tên trong Hook lại luôn bắt đầu bằng `use`. Chúng ta sẽ tìm hiểu trong [quy tắc trong Hooks](/docs/hooks-rules.html).

## Đọc State {#reading-state}

Khi bạn muốn hiển thị giá trị của `count` trong class, bạn dùng `this.state.count`:

```js
  <p>You clicked {this.state.count} times</p>
```

Trong function, chúng ta chỉ cần gọi trực tiếp biến count:


```js
  <p>You clicked {count} times</p>
```

## Cập nhật State {#updating-state}

Trong class, chúng ta cần gọi `this.setState()` để cập nhật lại giá trị cho `count`:

```js{1}
  <button onClick={() => this.setState({ count: this.state.count + 1 })}>
    Click me
  </button>
```

Trong function, chúng ta đã có `setCount` và không cần `this`:

```js{1}
  <button onClick={() => setCount(count + 1)}>
    Click me
  </button>
```

## Tóm tắt {#recap}

Bây giờ, chúng ta **tóm tắt lại những gì chúng ta đã học** và kiểm tra lại kiến thức.

<!--
  I'm not proud of this line markup. Please somebody fix this.
  But if GitHub got away with it for years we can cheat.
-->
```js{1,4,9}
 1:  import React, { useState } from 'react';
 2:
 3:  function Example() {
 4:    const [count, setCount] = useState(0);
 5:
 6:    return (
 7:      <div>
 8:        <p>You clicked {count} times</p>
 9:        <button onClick={() => setCount(count + 1)}>
10:         Click me
11:        </button>
12:      </div>
13:    );
14:  }
```

* **Dòng 1:** Chúng ta nhập `useState` Hook từ React. Nó cho phép giữ state cục bộ trong function component
* **Dòng 4:** Trong ví dụ, chúng tôi khai báo biến state bằng cách gọi `useState` Hook. Nó trả về một cặp giá trị do chúng ta đặt tên. Chúng tôi gọi là `count` bởi vì nó lưu trữ số lần nhấn nút. Chúng ta khởi tạo nó có giá trị là `0` bằng cách chuyển `0` thành đối số tring `useState`. Nó cho phép chúng ta cập nhật `count ` vì thế đặt tên nó thành `setCount`.
* **Dòng 9:** Khi người dùng nhấn vào button, chúng ta gọi `setCount` với một giá trị mới. React sau đó sẽ render ra Example  component và chuyển giá trị `count` mới cho nó.

Điều này có vẻ rất khó khi mới tiếp cận. Nhưng đừng bỏ cuộc, hãy bình tĩnh đọc lại các đoạn mã một lần nữa. Chúng tôi tin bạn!

### Mẹo nhỏ: Ý nghĩa của Square Brackets? {#tip-what-do-square-brackets-mean}

Bạn có nhớ dấu ngoặc vuông khi chúng ta khai báo biến state chứ:

```js
  const [count, setCount] = useState(0);
```

The names on the left aren't a part of the React API. You can name your own state variables:
Các tên trong dấu ngoặc vuông có thể được đặt theo ý thích của bạn:

```js
  const [Sơn, Tùng] = useState('đạo_nhạc');
```

Cú pháp Javascript này được gọi là ["array destructuring"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring). Điều đó có nghĩa là chúng ta tạo ra hai biến mới là `Sơn` và `Tùng` trong đó `Sơn` là giá trị đầu được trả về bởi `useState`, và `Tùng` là biến thứ 2. Nó tương đương với: 

```js
  var SơnTùng = useState('đạo_nhạc'); // Returns a pair
  var Sơn = SơnTùng[0]; // First item in a pair
  var Tùng = SơnTùng[1]; // Second item in a pair
```

khi chúng ta khai báo biến state với `useState`, nó sẽ trả về một cặp - một mảng gồm 2 phần tử. Phần tử đầu tiên chính là giá trị hiện tại, và phần tử thứ 2 là function cho phép cập nhật nó. Sử dụng `[0]` và `[1]` để truy cập cũng hơi khó hiểu, vì chúng có ý nghĩa cụ thể. Đó là lý do tại sao chúng ta sử dụng "array destructuring"

>Chú ý
>
> Bạn có thắc mắc làm thế nào mà React biết conponent nào `useState` tương ứng mà không sử dụng `this`. Chúng tôi đã [trả lời câu hỏi này](/docs/hooks-faq.html#how-does-react-associate-hook-calls-with-components) và nhiều câu hỏi khác trong mục FAQ.

### Mẹo nhỏ: Sử dụng nhiều biến state {#tip-using-multiple-state-variables}

Khai báo các biến state giống như `[something, setSomething]` vì nó rất tiện lợi. Cho phép chúng ta đặt tên khác nhau cho các state nếu chúng ta muốn sử dụng nhiều state:

```js
function ExampleWithManyStates() {
  // Declare multiple state variables!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
```

Trong component trên, chúng ta có `age`, `fruit`, và `todos` là các biến local và có thể cập nhật chúng riêng lẻ:

```js
  function handleOrangeClick() {
    // Similar to this.setState({ fruit: 'orange' })
    setFruit('orange');
  }
```

Bạn không cần sử dụng nhiều biến state. Các biến state có thể giữ các object và array khá tốt, vì vậy, chúng ta có thể nhóm các dữ liệu liên quan lại với nhau. Tuy nhiên, không giống như `this.setState` trong một class, cập nhật một state luôn thay thế thay vì hợp nhất nó.

[Các câu hỏi thường gặp](/docs/hooks-faq.html#should-i-use-one-or-many-state-variables).

## Bước tiếp theo {#next-steps}

Ở phần này, chúng ta tìm hiểu một trong những Hooks được cung cấp bởi React, được gọi là `useState`. Đôi khi chúng tôi gọi là "State Hook". Nó cho phép chúng ta thêm state vào các React function components - Như bước đầu chúng ta đã làm

Chúng ta đã tìm hiểu một chút về những gì Hook làm. Hook là các functions cho phép bạn "hook into" vào các tính năng của React
từ các function components. Chúng ta luôn đặt tên bắt đầu bằng `use` và có nhiều Hooks mà chúng ta chưa biết

**Tiếp tục [tìm hiểu Hook tiếp thep: `useEffect`.](/docs/hooks-effect.html)** It lets you perform side effects in components, and is similar to lifecycle methods in classes.
** Cho phép bạn thực hiện side effects trong components, và tương tự lifecycle methods trong class.
