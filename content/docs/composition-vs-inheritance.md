---
id: composition-vs-inheritance
title: Kết hợp và kế thừa
permalink: docs/composition-vs-inheritance.html
redirect_from:
  - "docs/multiple-components.html"
prev: lifting-state-up.html
next: thinking-in-react.html
---

React là kiểu kết hợp mạnh mẽ và chúng tôi khuyên bạn nên sử dụng kết hợp(composition) thay vì kế thừa(inheritance) giữa các component.

Trong phần này, chúng ta sẽ xem xét một vài vấn đề mà các developer mới sử dụng React sử dụng kế thừa(inheritance) và chỉ ra cách chúng ta có thể giải quyết chúng bằng kết hợp(composition)

## Chính sách ngăn chặn {#containment}

Một số component không biết phần con(children) của chúng. Điều này đặc biệt phổ biến đồi với các component như `Sidebar` hoặc `Dialog` là kết quả của `boxers`.

Chúng tôi khuyên bạn, với các component như thế nên sử dụng `children` prop để truyền trực tiếp các phần tử đầu vào của chúng:

```js{4}
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

Điều này cho phép các component khá truyền thành phần con tuỳ ý cho chúng bằng code JSX:

```js{4-9}
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

**[Try it on CodePen](https://codepen.io/gaearon/pen/ozqNOV?editors=0010)**

Mọi thứ bên trong thẻ JSX `<FancyBorder>` đều được chuyển vào component `<FancyBorder>` như một `children` prop. Khi `FancyBorder` render `{props.children}` bên trong `<div>`, các element được truyền vào sẽ hiển thị trong output.

Dù điều này không phổ biến, đôi khi bạn có thể cần nhiều "hole" như thế trong component. Trong những trường hợp như vậy, bạn có thể đưa ra quy ước riêng của mình thay vì `children`:

```js{5,8,18,21}
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/gwZOJp?editors=0010)

Các phần tử React như `<Contacts />` và `<Chat />` chỉ là các object, vì vậy bạn có thể truyền chúng dưới dạng props như mọi dữ liệu khác. Cách tiếp cận này có thể nhắc nhở bạn về các "slot" của người dùng trong các thư viện khác nhưng không giới hạn, bạn có thể sử dụng như props trong React.

## Chuyên sâu {#specialization}

Đôi khi chúng phải tư duy về các component như là trường hợp đặc biệt (special case) của các component khác. Ví dụ, chúng ta có thể nói rằng `WelcomeDialog` là trường hợp đặc biệt của `Dialog`.

Trong React, điều này cũng làm được bằng cách tự tạo ra, trong đó, một component xác định cụ thể sẽ hiển thị một component chung chung và một cấu hình với các props:

```js{5,8,16-18}
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/kkEaOZ?editors=0010)

Component hoạt động như nhau trong các component được định nghĩa như class:

```js{10,27-31}
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login}
               onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/gwZbYa?editors=0010)

## Kế thừa thì sao? {#so-what-about-inheritance}

Tại Facebook, chúng tôi sử dụng React trong hàng ngàn thành phần và chúng tôi đã tìm thấy bất cứ trường hợp nào sử dụng nào đó, chúng tôi khuyên bạn nên tạo cấu trúc phân cấp kế thừa thành phần.

Props và sự kết hợp cung cấp cho bạn sự linh hoạt mà bạn cần để tuỳ chỉnh giao diện và hành vi của component theo cách rõ ràng và an toàn. Hãy nhớ rằng các thành phần có thể chấp nhận các props tuỳ ý, bao gồm các giá trị nguyên thuỷ, React element hoặc các function.

Nếu bạn muốn tái sử dụng các non-UI function giữa các component, chúng tôi khuyên bạn nên trích xuất(extracting) nó thành một module Javascript. Các component có thể nhập(import) và sử dụng function, object hoặc class mà không cần mở rộng nó.