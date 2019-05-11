---
id: hooks-state
title: Using the Effect Hook
permalink: docs/hooks-effect.html
next: hooks-rules.html
prev: hooks-state.html
---

*Hooks* là một tính năng mới được thêm vào từ phiên bản 16.8. Cho phép sử dụng state và các tính năng React mà không cần viết class

*Effect Hook* cho phép bạn xử lý các side effects trong function components:

```js{1,6-10}
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

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

Đoạn mã trên được lấy từ [ví dụ của phần trước](/docs/hooks-state.html), nhưng đã được thêm tính năng: chúng tôi đặt lại tiêu đề trang một tin nhắn khi click vào button.

Lấy dữ liệu và hiển thị, thiết lập đăng ký, và thay đổi DOM trong React components là tất cả các ví dụ về thực thi "side effects" (hoặc đơn giản là "effects"), cho dù bạn có hay không sử dụng để gọi các hoạt động này trong component hay không.

>Mẹo nhỏ
>
>Nếu bạn quen thuộc với React class lifecycle methods, bạn có thể nghĩ đến `useEffect` Hook kết hơp `componentDidMount`, `componentDidUpdate` và `componentWillUnmount`.

Có hai loại side effect phổ biến trong React components: Những loại không có yêu cầu dọn dẹp, và những loại yêu cầu. Hãy cùng xem chi tiết sự khác biệt này nào...

## Effects không dọn dẹp {#effects-without-cleanup}

Đôi khi chúng ta muốn **chạy một số mã bổ sung sau khi React cập nhật lại DOM**  khi có request, các DOM sẽ thay đổi và ghi lại log trong Network là ví dụ phổ biến về effrcts không yêu cầu dọn dẹp. Chúng ta có thể nói rằng có thể chạy chúng mà không cần quan tâm chúng. Và giờ hãy so sánh các class và Hooks thể hiện điều này.
### Ví dụ sử dụng Classes {#example-using-classes}

Trong React class components, phương thức `render` không gây ra side effects. Nó quá sớm để thực hiện, mà chúng tôi muốn thực hiện các effect *sau khi* React đã cập nhật lại DOM.

Đó là lý do tại sao trong các React class, chúng tôi lại đặt side effect vào trong `componentDidMount` và `componentDidUpdate`. Quay trở lại ví dụ, đây là React counter class component cập nhật lại title ngay sau khi React thay đổi DOM:

```js{9-15}
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
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

Lưu ý **chúng ta tạo hai dòng lệnh giống nhau giữa hai phương thức lifecycle methods trong class**

Đó là do trong nhiều trường hợp, chúng ta muốn thực hiện nhũng side effect giống nhau bất kể các components được gắn hoặc cập nhật. Về mặt khái niệm, chúng tôi muốn nó thực hiện sau mỗi lần `render` - nhưng React class components không có các phương thức như thế. Chúng ta có thể tạo ra 2 phương thức riêng biệt nhưng sẽ vẫn gọi nó ở 2 nơi.

Bây giờ hãy xem chúng ta có hãy xem `useEffect` Hook có thể làm được gì nào.

### Ví dụ sử dụng Hooks {#example-using-hooks}

Chúng ta đã thấy ví dụ này trên đầu trang, nhưng hãy phân tích lại nó:

```js{1,6-8}
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

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

**Tác dụng của `useEffect` là gì?** Bằng cách sử dụng Hook, chúng ta nói với React rằng component của chúng ta cần làm gì sau khi `render`. React sẽ ghi nhớ những lần mà function của chúng ta đã vượt qua(chúng tôi gọi nó là "effect"), gọi nó sau khi thực hiện cập nhật lại DOM. Trong effect này, chúng ta gán lại giá trị cho document.title, nhưng cũng có thể thực hiện nạp dữ liệu hoặc gọi một số API.

**Tại sao ``useEffect` được gọi bên trong một component** Đặt `useEffect` bên trong component cho phép chúng ta truy cập vào biến `count`(hoặc bất kỳ props nào) ngay từ effect. Chúng ta không cần một API đặc biệt nào để đọc nó, vì nó đã có trong function scope. Hook bao quanh Javascript và tránh đem vào những API React đặc biệt bởi vì Javascript đã cung cấp những giải pháp rồi.

**`useEffect` có chạy sau mỗi lần render hay không?**  Có, và nó mặc chạy sau lần render đầu tiên, và sau mỗi lần cập nhật. (Chúng ta sẽ nói sau về [cách cấu hình lại chúng](#tip-optimizing-performance-by-skipping-effects).) Thay vì suy nghĩ về các khía cạnh của việc "mounting" và "updating", bạn có thể dễ dàng hơn khi nghĩ rằng các effect sảy ra "sau khi render". React đảm bảo rằng DOM đã được cập nhật vào thời điểm nó chạy các effect 

### Chi tiết {#detailed-explanation}

Bây giờ chúng ta đã hiểu hơn về effect và những dòng code này: 

```js
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

Chúng ta khai báo biến `count` và nói với React rằng cần sử dụng một effect. Chúng ta chuyển chức năng đó cho `useEfect` Hook. function này chứa effect. Bên trong effect, chúng ta gán giá trị cho document title bằng API của trình duyệt `document.title`. Chúng ta có thể đọc biến `count` mới nhất trong effect bởi nó trong phạm vi scope của function. Khi React renders các component của chúng ta, nó sẽ ghi nhớ các effect mà chúng ta sử dụng, và sau đó chạy những effect này sau khi cập nhật lại DOM. Điều này diễn ra ở mọi lần render, ngay cả ở lần render đầu tiên

Những developer Javascript có kinh nghiệm có thể thấy rằng function truyền cho `useEffect` sẽ khác nhau sau mỗi lần render. Điều này là cố ý. Trong thực tế, đây là những gì cho phép chúng ta đọc giá trị biến `count` từ trong effect mà không lo nó không cập nhật lại giá trị mỗi khi chúng ta render lại, chúng tôi ghi lại cho mỗi effect khác nhau, thay thế chúng cho các effect trước đó. Nói cách khác, điều này làm cho các effect hoạt động giống như một phần của kết quả render - mỗi effect thuộc về một render cụ thể. Chúng ta sẽ thấy rõ hơn tại sao điều này hữu ích sau khi đọc đến [phần này](#explanation-why-effects-run-on-each-update).

>Mẹo nhỏ
>
>Không giống như `componentDidMount` hoặc `componentDidUpdate`, effect sẽ lên kế hoặc với `useEffect` không chặn trình duyệt khi màn hình đang cập nhật. Điều này làm cho cảm giác chương trình chạy nhanh hơn. Phần lớn, các effect không cần chạy đồng bộ. Trong một số trường hợp(chẳng hạn như đo bố cục), có một `useLayoutEffect` Hook đặc biệt với API giống hệt với `useEffect`.

## Effects yêu cầu làm sạch {#effects-with-cleanup}

Trước đó, chúng ta đã tìm hiểu effect không yêu cầu làm sạch, Tuy nhiên một số effect lại yêu cầu. Ví dụ, **chúng ta muốn thiết lập đăng ký** một số nguồn dữ liệu bên ngoài. Trong trường hợp đó, điều quan trọng là phải làm sạch để không bị tràn bộ nhớ, Hãy so sánh các cách chúng ta có thể làm với class và Hooks.


### Sử dụng Classes {#example-using-classes-1}

Trong React class, thông thường, bạn sẽ thiết lập đăng ký trong `componentDidMount` và làm sạch nó trong `componentWillUnmount`.  Ví dụ, chúng ta có module ChatAPI lấy trạng thái bạn bè online. Ở đây, chúng ta có thể thiết lập, và hiển thị trạng thái online bằng class:

```js{8-26}
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }
    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```

Lưu ý `componentDidMount` và `componentWillUnmount` cần đối ngược nhau. Các phương thức lifecycle buộc chúng ta phải phân tách logic này, mặc dù nội dung trong cả hai đều là một effect. 

>Chú ý
>
>Bạn đọc tinh ý có thể nhận thấy rằng ví dụ chính xác cũng cần thêm `componentDidUpdate`. Chúng tôi đã loại bỏ nó, nhưng sẽ quay lại nó trong [phần sau](#explanation-why-effects-run-on-each-update)

### Sử dụng Hooks {#example-using-hooks-1}

Hãy xem chúng ta có thể làm được gì với Hooks.

Bạn có nghĩ rằng chúng ta cần thiết lập một effect riêng để thực hiện việc làm sạch. Nhưng code để thêm và xoá đăng ký phải liên quan chặt chẽ đvì thế `useEffect` được thết kế để thực hiện chúng cùng nhau. Nếu effect của bạn trả về một function, React sẽ chạy nó khi đến lúc dọn dẹp:

```js{6-16}
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

**Tại sao lại return function trong effect?** Đây là cơ chế làm sạch cho các effect. Mỗi effect có thể trả về một chức năng dọn dẹp sau nó. Điều này cho phép chúng ta giữ logic để thêm và xoá đăng ký gần nhau. Nó là nhóm các effect tương tự nhau

**Chính xác thì khi nào React dọn dẹp một effect?** React thực hiện việc làm sạch khi component ngắt kết nối. Tuy nhiên, như chúng ta đã tìm hiểu trước đó, các effect chạy trong mọi lần render. Đây là lý do tại sao React cũng làm sạch các effect từ lần render trước đó khi chạy effect vào lần tiếp theo. Chúng ta sẽ thảo luận về lý do [tại sao điều này tránh gặp bugs](#explanation-why-effects-run-on-each-update) và [làm thế nào để tăng hiệu suất](#tip-optimizing-performance-by-skipping-effects) ngay sau đây.

>Chú thích
>
>Chúng ta không phải return tên của một function từ effect. Chúng tôi đã gọi nó là `dọn dẹp` để làm rõ mục đích của nó, nhưng bạn có thể return một arrow function hoặc gọi nó bằng một cái tên khác.

## Tóm tắt {#recap}

Chúng ta đã tìm hiểu được rằng `useEffect` cho phép chúng ta thể hiện các effect khác nhau sau mỗi lần render. Một số effect có thể yêu cầu làm sạch để chúng return một function:

```js
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

Một số effect khác có thể không có giai đoạn dọn dẹp và không cần return gì.

```js
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

Trong 2 trường hợp Effect Hook sử dụng với một API.  

-------------

**Nếu bạn cảm thấy mình hiểu được cách thức hoạt động của Effect Hook, bạn có thể chuyể sang [trang tiếp theo, nói về các quy tắc trong Hooks](/docs/hooks-rules.html) ngay bây giờ**

-------------

## Một số mẹo cho việc sử dụng các Effect {#tips-for-using-effects}

Chúng ta sẽ tìm hiểu tiếp với một cái nhìn sâu hơn về một số khía cạnh của `useEffect` mà một số người đã có kinh nghiệm quan tâm. Bạn không cần phải bắt buộc tìm hiểu sâu về chúng ngay bây giờ, mà có thể quay lại trang này để tìm hiểu sâu hơn về Effect Hook vào bất cứ lúc nào.

### Mẹo: Sử dụng nhiều Effect riêng biệt {#tip-use-multiple-effects-to-separate-concerns}

Một trong những vấn đề chúng ta đã [trình bày](/docs/hooks-intro.html#complex-components-become-hard-to-understand) trong Hooks là các lifecycle của class thường chứa logic không liên quan, nhưng logic liên quan bị chia thành nhiều phương thức. Đây là một component kết hợp đếm và lấy trạng thái bạn bè online từ các ví dụ trước:

```js
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
```

Note how the logic that sets `document.title` is split between `componentDidMount` and `componentDidUpdate`. The subscription logic is also spread between `componentDidMount` and `componentWillUnmount`. And `componentDidMount` contains code for both tasks.
Lưu ý, thời điểm để gán `document.title` là khoảng thời gian `componentDidMount` và `componentDidUpdate`. logic đăng ký cũng được nằm trong khoảng thời gian

So, how can Hooks solve this problem? Just like [you can use the *State* Hook more than once](/docs/hooks-state.html#tip-using-multiple-state-variables), you can also use several effects. This lets us separate unrelated logic into different effects:

```js{3,8}
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

**Hooks lets us split the code based on what it is doing** rather than a lifecycle method name. React will apply *every* effect used by the component, in the order they were specified.

### Explanation: Why Effects Run on Each Update {#explanation-why-effects-run-on-each-update}

If you're used to classes, you might be wondering why the effect cleanup phase happens after every re-render, and not just once during unmounting. Let's look at a practical example to see why this design helps us create components with fewer bugs.

[Earlier on this page](#example-using-classes-1), we introduced an example `FriendStatus` component that displays whether a friend is online or not. Our class reads `friend.id` from `this.props`, subscribes to the friend status after the component mounts, and unsubscribes during unmounting:

```js
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
```

**But what happens if the `friend` prop changes** while the component is on the screen? Our component would continue displaying the online status of a different friend. This is a bug. We would also cause a memory leak or crash when unmounting since the unsubscribe call would use the wrong friend ID.

In a class component, we would need to add `componentDidUpdate` to handle this case:

```js{8-19}
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate(prevProps) {
    // Unsubscribe from the previous friend.id
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // Subscribe to the next friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
```

Forgetting to handle `componentDidUpdate` properly is a common source of bugs in React applications.

Now consider the version of this component that uses Hooks:

```js
function FriendStatus(props) {
  // ...
  useEffect(() => {
    // ...
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

It doesn't suffer from this bug. (But we also didn't make any changes to it.)

There is no special code for handling updates because `useEffect` handles them *by default*. It cleans up the previous effects before applying the next effects. To illustrate this, here is a sequence of subscribe and unsubscribe calls that this component could produce over time:

```js
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // Run first effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // Run next effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // Run next effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Clean up last effect
```

This behavior ensures consistency by default and prevents bugs that are common in class components due to missing update logic.

### Tip: Optimizing Performance by Skipping Effects {#tip-optimizing-performance-by-skipping-effects}

In some cases, cleaning up or applying the effect after every render might create a performance problem. In class components, we can solve this by writing an extra comparison with `prevProps` or `prevState` inside `componentDidUpdate`:

```js
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

This requirement is common enough that it is built into the `useEffect` Hook API. You can tell React to *skip* applying an effect if certain values haven't changed between re-renders. To do so, pass an array as an optional second argument to `useEffect`:

```js{3}
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

In the example above, we pass `[count]` as the second argument. What does this mean? If the `count` is `5`, and then our component re-renders with `count` still equal to `5`, React will compare `[5]` from the previous render and `[5]` from the next render. Because all items in the array are the same (`5 === 5`), React would skip the effect. That's our optimization.

When we render with `count` updated to `6`, React will compare the items in the `[5]` array from the previous render to items in the `[6]` array from the next render. This time, React will re-apply the effect because `5 !== 6`. If there are multiple items in the array, React will re-run the effect even if just one of them is different.

This also works for effects that have a cleanup phase:

```js{10}
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // Only re-subscribe if props.friend.id changes
```

In the future, the second argument might get added automatically by a build-time transformation.

>Note
>
>If you use this optimization, make sure the array includes **all values from the component scope (such as props and state) that change over time and that are used by the effect**. Otherwise, your code will reference stale values from previous renders. Learn more about [how to deal with functions](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) and [what to do when the array changes too often](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often).
>
>If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array (`[]`) as a second argument. This tells React that your effect doesn't depend on *any* values from props or state, so it never needs to re-run. This isn't handled as a special case -- it follows directly from how the inputs array always works.
>
>If you pass an empty array (`[]`), the props and state inside the effect will always have their initial values. While passing `[]` as the second argument is closer to the familiar `componentDidMount` and `componentWillUnmount` mental model, there are usually [better](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) [solutions](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often) to avoid re-running effects too often. Also, don't forget that React defers running `useEffect` until after the browser has painted, so doing extra work is less of a problem.
>
>We recommend using the [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) rule as part of our [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) package. It warns when dependencies are specified incorrectly and suggests a fix.

## Next Steps {#next-steps}

Congratulations! This was a long page, but hopefully by the end most of your questions about effects were answered. You've learned both the State Hook and the Effect Hook, and there is a *lot* you can do with both of them combined. They cover most of the use cases for classes -- and where they don't, you might find the [additional Hooks](/docs/hooks-reference.html) helpful.

We're also starting to see how Hooks solve problems outlined in [Motivation](/docs/hooks-intro.html#motivation). We've seen how effect cleanup avoids duplication in `componentDidUpdate` and `componentWillUnmount`, brings related code closer together, and helps us avoid bugs. We've also seen how we can separate effects by their purpose, which is something we couldn't do in classes at all.

At this point you might be questioning how Hooks work. How can React know which `useState` call corresponds to which state variable between re-renders? How does React "match up" previous and next effects on every update? **On the next page we will learn about the [Rules of Hooks](/docs/hooks-rules.html) -- they're essential to making Hooks work.**
