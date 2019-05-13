---
id: state-and-lifecycle
title: State and Lifecycle
permalink: docs/state-and-lifecycle.html
redirect_from:
  - "docs/interactivity-and-dynamic-uis.html"
prev: components-and-props.html
next: handling-events.html
---

Page này giới nói về các khái niệm của state và lifecycle trong React component. Bạn có thể xem thêm [chi tiết component API reference tại đây](/docs/react-component.html).

Cùng nhìn lại ví dụ về đồng hồ trong [các phần trước](/docs/rendering-elements.html#updating-the-rendered-element). trong [Rendering các phần tử](/docs/rendering-elements.html#rendering-an-element-into-the-dom), chúng ta chỉ được học cách để cập nhật UI. Chúng ta gọi `ReactDOM.render()` để render các kết quả:

```js{8-11}
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/gwoJZk?editors=0010)

Trong phần này, chúng ta sẽ tìm hiểu cách làm cho `Clock` component có thể đóng gói và tái sử dụng. Nó sẽ thiết lập bộ đếm thời gian và tự cập nhật nó mỗi giây.

Chúng ta có thể viết gộp vào như sau:

```js{3-6,12}
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/dpdoYR?editors=0010)

Ideally we want to write this once and have the `Clock` update itself:
Lý tưởng nhất là chúng tôi muốn viết cái này một lần và nó tự cập nhật lại `Clock`.

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Để thực hiện điều này, chúng ta cần thêm `state` cho `Clock` component.

State tương tự như props, nhưng nó private và được kiểm soát bởi component.

## Chuyển đổi một Function sang một Class {#converting-a-function-to-a-class}

Bạn có thể chuyển đổi một function component `Clock` thành một class bằng 5 bưới sau:

1. Tạo một [ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes), có cùng tên và extends `Reac.Component()`

2. Thêm một phương thức vào, gọi là "render()".

3. Chuyển phần nội dung của function vào trong phương thức `render()`.

4. Đổi `props` thành `this.props` trong `render()`.

5. Xoá các khai báo còn lại trong hàm.

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/zKRGpo?editors=0010)

`Clock` bây giờ được định nghĩa là class chứ không còn là function.

Phương thức `render()` sẽ được gọi mỗi lần có cập nhật, nhưng miễn là chúng ta render `<Clock />` vào cùng một DOM node, chỉ một thể hiện duy nhất của `Clock` được sử dụng. Điều này cho phép chúng ta sử dụng các tính nằng bổ sung như local state và phương thức lifecycle.

## Thêm Local State vào Class {#adding-local-state-to-a-class}

Chúng ta sẽ chuyển `date` từ props sang state trong 3 bước:

1) Thay `this.props.date` bằng `this.state.date` trong phương thức `render()`:

```js{6}
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

2) Thêm [class constructor](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes#Constructor) tạo giá trị ban đầu cho `this.state`:

```js{4}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Lưu ý, chúng tôi đặt `props` vào làm base(cơ sở) contructor:

```js{2}
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

Class component luôn luôn gọi base contructor bằng `props`.

3) bỏ `date` prop trong phần tử `<Clock />`:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Chúng ta sẽ thêm phần hẹn giờ để trở lại chính component đó.

Kết quả sẽ trông như thế này:

```js{2-5,11,18}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/KgQpJd?editors=0010)

Tiếp theo, chúng ta sẽ thiết lập bộ đếm giờ cho `Clock` và tự cập nhật mỗi giây. 

## Thêm phương thức Lifecycle cho Class {#adding-lifecycle-methods-to-a-class}

Trong các ứng dụng có nhiều component, điều này rất quan trọng để giải phóng tài nguyên được lấy bởi component kho chúng bị phá huỷ.

Chúng ta muốn [thiết lập bộ đếm](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) bất cứ khi nào `Clock` được render DOM lần đầu tiên. Trong React, điều này được gọi là "mouting" .

Chúng ta cũng muốn [xoá bộ đếm](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval) bất cứ khi nào DOM do `Clock` tạo ra bị xoá. Điều này trong React gọi là "unmouting".

Chúng ta có thể khai báo các phương thức đặc biệt trên component class để chạy một số code khi component "mount" và "unmount".

```js{7-9,11-13}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {

  }

  componentWillUnmount() {

  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Những phương thức này gọi là "vòng đời"(lifecycle method).

Phương thức `componentDidMount()` chạy sau khi component render ra DOM. Đây là một nơi tốt để thiết lập bộ đếm:

```js{2-5}
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

Lưu ý cách chúng ta lưu timer ID trên `this`.

Mặc dù `this,props` được thiết lập bởi React và `this.state` có ý nghĩa đặc biệt, bạn có thể tự do thêm các trường (field) vào trong class nếu bạn cần lưu trữ cái gì đó tham gia luồng dữ liệu (ở ví dụ này là timerID).

Chúng ta sẽ phá bỏ bộ đếm trong `componentWillUnmount()` lifecycle method:

```js{2}
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

Cuối cùng, chúng ta sẽ thực hiện một phương thức để gọi `tick()` mà `Clock` component sẽ chạy mỗi giây.

Nó sẽ sử dụng `this.setState()` để thiết lập cập nhật cho component local state:

```js{18-22}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/amqdNA?editors=0010)

Bây giờ thì đồng hồ đã chạy.

Tóm tắt lại những gì đã diễn ra ở trên:

1) Khi `<Clock />` chạy đến `ReactDOM.render()`, React gọi contructor của component `Clock`.Vì `Clock` cần hiển thị thời gian hiện tại, nên nó khởi tạo `this.state` với một object là thời gian hiện tại. Chúng ta sẽ cập nhật state này sau.

2) React gọi tới phương thức `render()` của  component `Clock`. Đây là lúc React biết những gì sẽ được hiển thị ra màn hình. React sau đó cập nhật lại DOM để khớp với đầu ra render của `Clock`.

3) Khi đầu ra `Clock` được chèn vào trong DOM, React gọi phương thức lifecycle `componentDidMount()`. Bên trong nó, component `Clock` yêu cầu trình duyệt để thiết lập lại bộ đếm bằng cách gọi lại phương thức `tick` của component mỗi sau mỗi giây.

4) Mỗi giây, trình duyệt gọi phương thức `tick()` một lần. Trong nó, component `Clock` lên lịch cập nhật UI bằng cách gọi `setState()` với object chứa thời gian hiện tại. Nhờ `setState()` được gọi, React hiểu rằng state đã thay đổi, và gọi phương thức `render()` một lần nữa để in ra màn hình những thay đổi. Lần này `this.state.date` trong phương thức `render()` sẽ khác và do đó, đầu ra của render sẽ là thời gian đã được cập nhật. React cập nhật lại DOM cho phù hợp. 

5) Nếu component `Clock` bị xoá khỏi DOM, React sẽ gọi phương thức lifecycle `componentWillUnmount()` để dừng lại bộ đếm.

## Sử dụng đúng State {#using-state-correctly}

Có vài thứ bạn nên hiểu và `setState()`.

### Không sửa đổi trực tiếp State {#do-not-modify-state-directly}

Ví dụ: điều này sẽ không render lại component:

```js
// Wrong
this.state.comment = 'Hello';
```

Thay vào đó, hãy sử dụng `setState()`:

```js
// Correct
this.setState({comment: 'Hello'});
```

Vị trí duy nhất mà bạn có thể gán `this.state` chính là contructor.

### State được cập nhật có thể bất đồng bộ {#state-updates-may-be-asynchronous}

React có thể gộp nhiều `setState()` cho một lần cập nhật duy nhất để đảm bảo hiệu suất.

Bởi vì `this.props` và `this.state` có thể được cập nhật giá trị bất đồng bộ, bạn không nên dựa vào các giá trị của chúng để tính toán các state tiếp theo.

Ví dụ: Code dưới đây có thể không cập nhật lại bộ đếm:

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

Để sửa nó, sử dụng dạng thứ 2 của `setState()` chấp nhận một function chứ không phải là một object. Function đó sẽ nhận state trước đó làm đối số thứ nhất, và props tại thời điểm cập nhật được áp dụng làm đối số thứ hai:

```js
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

Chúng ta sử dụng [arrow function](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions), nhưng nó cũng hoạt động với những function bình thường:

```js
// Correct
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

### Hợp nhất cập nhật State {#state-updates-are-merged}

Khi bạn gọi `setState()`, React sẽ hợp nhất object mà bạn cung cấp vào state hiện tại.

Ví dụ: State của bạn có thể chứa một số biến cô lập:

```js{4,5}
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```

Sau đó, bạn có thể cập nhật chúng độc lập bằng cách gọi `setState()`:

```js{4,10}
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

Việc hợp nhất là không sâu, vì thế `this.setState({comments})` được cập nhật mà `this.state.posts` được giữ nguyên.

## Luồng dữ liệu {#the-data-flows-down}

Cả các thành phần cha và con đều không biết liệu compnent có là stateful hay stateless, và chúng không nên quan tâm nó được được định nghĩa là function hay class.

Đó là lý do tại sao state thường được gọi là local hoặc encapsulated. Không component nào khác có thể truy cập và gán giá trị được vào nó.

Một component có thể chọn chuyển state làm props cho các child component:

```js
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```

Điều này cũng hoạt động cho các component do người dùng định nghĩa:

```js
<FormattedDate date={this.state.date} />
```

Component `FormattedDate` sẽ nhận được `date` trong props và nó sẽ không biết nó được lấy từ `Clock` state hay từ `Clock` props, hay được gõ bằng tay:

```js
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/zKRqNB?editors=0010)

Điều này thường được gọi là "top-down" hoặc luồng dữ liệu "unidirectional". Bất kỳ state nào được sở hữu bởi component, và bất kì dữ liệu hoặc UI có nguồn gốc từ state đó chỉ có thể ảnh hường đến các component bên dưới.

Nếu bạn tưởng tượng một component tree như một thác nước props, thì mỗi state giống như một nguồn nước bổ sung tại một thời điểm tuỳ ý và cũng đi từ trên xuống.

Để thấy tất cả các component độc lập nhau, chúng ta có thể tạo một component `App` và render ra 3 `<Clock />`:

```js{4-6}
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/vXdGmd?editors=0010)

Mỗi `Clock` được thiết lập bộ đếm riêng và cập nhật độc lập nhau.

Trong các ứng dụng React, việc một component có statefull hay stateless được coi là một chi tiết của component có thể thay đổi theo thời gian. Bạn có thể sử dụng các stateless component bên trong các stateful component và ngược lại. 
