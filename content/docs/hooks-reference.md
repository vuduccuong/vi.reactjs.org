---
id: hooks-reference
title: Hooks API Reference
permalink: docs/hooks-reference.html
prev: hooks-custom.html
next: hooks-faq.html
---

*Hooks* mới được thêm ở phiên bản React 16.8. Cho phép bạn sử dụng state và các chức năng khác của React mà không cần tạo class.

Trang này mô tả các API cho Hooks tích hợp trong React.

Nếu bạn mới sử dụng Hooks, bạn có thể muốn xem [tổng quan](/docs/hooks-overview.html) trước. Bạn cũng có thể tìm thấy thông tin hữu ích trong [câu hỏi thường gặp](/docs/hooks-faq.html)

- [Hooks cơ bản](#basic-hooks)
  - [`useState`](#usestate)
  - [`useEffect`](#useeffect)
  - [`useContext`](#usecontext)
- [Hooks nâng cao](#additional-hooks)
  - [`useReducer`](#usereducer)
  - [`useCallback`](#usecallback)
  - [`useMemo`](#usememo)
  - [`useRef`](#useref)
  - [`useImperativeHandle`](#useimperativehandle)
  - [`useLayoutEffect`](#uselayouteffect)
  - [`useDebugValue`](#usedebugvalue)

## Hooks cơ bản {#basic-hooks}

### `useState` {#usestate}

```js
const [state, setState] = useState(initialState);
```

Trả về một state mang giá trị, và một function cập nhật nó.

Trong lần render đầu tiên, state được return (`state`) giống với giá trị được truyền dưới dạng đối số đầu tiên (`initialState`).

function `setState` được sử dụng để cập nhật lại state. Nó chấp nhận một state với giá trị mới và ghi lại kết quả sau mỗi lần render của component.


```js
setState(newState);
```

Trong các lần render tiếp theo, giá trị đầu tiên được return bởi `useState` sẽ luôn là state gần nhất sau khi được cập nhật.


#### Functional updates {#functional-updates}

Nếu state mới được máy tính sử dụng state trước đó, bạn có thể sử dụng function `setState`. Function này sẽ nhận giá trị trước đó, và return giá trị được cập nhật. Dưới đây là ví dụ về counter component sử dụng  `setState`:

```js
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
    </>
  );
}
```

Nút "+" và "-" sử dụng function form, bởi vì giá trị cập nhật dựa trên giá trị trước đó. Nhưng nút "Reset" sử dụng đối số là giá trị ban đầu, bởi vì nó luôn đặt lại biến `count` về giá trị ban đầu.

> Chú thích
>
>
> Không giống như phương thức `setState` được tìm thấy trong class component, `useState` không dự động hợp cập nhật các object. Bạn có thể thay đổi behavior bằng cú pháp:
> ```js
> setState(prevState => {
>   // Object.assign would also work
>   return {...prevState, ...updatedValues};
> });
> ```
>
> Một tuỳ chọn khác là `useReducer`, phù hợp hơn để quản lý các state object có chứa nhiều sub-value.

#### Lazy initial state {#lazy-initial-state}

Đối số `initialState` được sử dụng trong lần render đầu tiên. Trong những lần render tiếp theo, nó bị bỏ qua. Nếu trạng thái ban đầu là kết quả của một tính toán, bạn có thể cung cấp một hàm thay thế, nó sẽ chỉ được thực hiện trên lần render ban đầu:

```js
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

#### Bailing out of a state update {#bailing-out-of-a-state-update}

Nếu bạn cập nhật State Hook về cùng giá trị với giá trị hiện tại, React sẽ giữ nguyên mà không render ra các children hoặc các effect. (React sử dụng [thuật toán `Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description).))

Lưu ý rằng React có thể vẫn cần render lại các component cụ thể nào đó trước khi bailing out. Đó không phải là một vấn đề đáng lo lắng vì React đã không "đi sâu" vào. Nếu bạn đang thực hiện các phép tính trong khi render, bạn có thể tối ưu chúng bằng `useMomo`.

### `useEffect` {#useeffect}

```js
useEffect(didUpdate);
```

Chấp nhận một function chứa mệnh lệnh, có thể giúp cải thiện code.

Mutations, subscriptions, timers, logging, and other side effects không được phép bên trong main body của function component. Làm như vậy sẽ dẫn đến lỗi và không nhất quán trong UI.

Thay vào đó, hãy sử dụng `useEffect`. Function được truyền cho `useEffect` sẽ chạy sau khi render ra màn hình.

Mặc định, các effect chạy sau khi render hoàn thành, nhưng bạn có thể chọn [giá trị nhất định khi đã thay đổi](#conditionally-firing-an-effect).

#### Cleaning up an effect {#cleaning-up-an-effect}

Thông thường các effect tạo ra các tài nguyên cần được dọn sạch trước khi component hoàn thành, chẳng han như subscription hoặc timer ID. Để thực hiện việc này, function được truyền cho `useEffect` có thể trả về clean-up function. Ví dụ: để tạo một subscription:

```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Clean up the subscription
    subscription.unsubscribe();
  };
});
```

Clean-up function chạy trước khi loại bỏ component khỏi UI để tránh memory leaks. Ngoài ra, nếu một component render nhiều lần (như chúng thường làm), **effect trước đó sẽ được xoá sạch trước khi thực hiện effect tiếp theo**. Trong ví dụ của chúng ta, điều này có nghĩa là đăng ký mới được tạo trên mỗi lần cập nhật, hãy tham khảo phần tiếp theo.

#### Timing of effects {#timing-of-effects}

Không giống như `componentDidMount` và `componentDidUpdate`, function dùng `useEffect` kích hoạt **sau khi** bố trí layout và vẽ trong một event bị trì hoãn. Điều này làm cho nó phù hợp với nhiều side effect dùng chung, giống như việc thiết lập đăng ký và sử lý event, bởi vì hầu hết các loại tác động không thể chặn trình duyệt cập nhật lại màn hình.

Tuy nhiên, không phải tất cả các effect có thể được hoãn lại. Ví dụ, một DOM hiển thị cho người dùng phải bắn(fires) đồng bộ(asynchronously) trước lần vẽ lại tiếp theo để người dùng nhận thấy sự không nhất quá trực quan. Đối với sự kiện này, React cung cấp thêm một Hook gọi là [`useLayoutEffect`](#uselayouteffect). Nó tương tự như `useEffect` và chỉ khác khi nó được triển khai.

Mặc dù `useEffect` được trì hoãn cho đễ khi trình duyệt được vẽ lại, nó vẫn đảm bảo sẽ kích hoạt trước khi có bất kì render mới nào. React sẽ luôn xoá một render effect trước đó khi bắt đầu một cập nhật mới.

#### Điều kiện để bắn một effect {#conditionally-firing-an-effect}

Hành vi(behavior) mặc định cho các effect là kích hoạt effect sau mỗi lần hoàn thành. Bằng cách đó, một effect luôn được tạo lại nếu một trong những phụ thuộc(dependencies) của nó thay đổi.

Tuy nhiên, điều này có thể là quá mức cần thiết trong một số trường hợp, giống như ví dụ ở lần trước. Chúng ta không cần phải tạo ra một đăng ký(subscription) mới trong mỗi lần cập nhật, chỉ khi `source` props đã thay đổi. 

Để thực hiện đaieeuf này, hãy truyền một tham số thứ hai cho `useEffect`, đó là mảng các giá trị mà effect phụ thuộc vào. Ví dụ cập nhật của chúng ta bây giờ trông như thế này:

```js
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```
Bây giờ, đăng ký(subscription) sẽ chỉ được tạo lại khi `props.source` thay đổi.

>Chú thích
>
>Nếu bạn sử dụng tối ưu hoá này, hãy đảm bảo mảng bao gồm **tất cả các giá trị từ component scope (chẳng hạn như props và state) thay đổi theo thời gian và được sử dụng bởi effect**. Nếu không, code của bạn sẽ tham chiếu các giá trị cũ từ các lần render trước đó. Tìm hiểu thêm về [các xử lý với các function](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) và phải làm gì khi [các giá trị mảng thay đổi quá thường xuyên](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often).
>
>Nếu bạn muốn chạy một effect và làm sạch nó chỉ một lần (trên mout và unmount), bạn có thể chuyển một mảng trống làm đối số thứ hai. Điều này nói với React rằng effect của bạn không phụ thuộc vào bất cứ giá trị nào của props hoặc state, do đó nó không bao giờ cần phải chạy lại. Điều này được xử lý như một trường hợp đặc biệt - nó diễn ra trực tiếp từ cách mảng phụ thuộc luôn hoạt động.
>
>Nếu bạn vượt qua(pass) một mảng trống (`[]`), các props và state bên trong effect sẽ luôn có các giá trị ban đầu của chúng. Trong khi chuyển `[]` là đối số thứ hai gần hơn với mô hình `componentDidMount` và `componentWillUnmount`, thường có các giải pháp tốt hơn để tránh các effect chạy lại thường xuyên. Ngoài ra, đừng quên rằng React trì hoãn việc sử dụng `useEffect` cho đến khi trình duyệt đã vẽ xong, do đó, việc làm thêm sẽ ít lỗi hơn.
>
>Chúng tôi khuyên bạn nên sử dụng [`exhaustive-deps`](https://github.com/facebook/react/issues/14920), nó là một phần của gói [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation). Nó cảnh báo khi các phụ thuộc được chỉ định không xác định và đề nghị sửa chữa.

Mảng phụ thuộc(dependencies) không được truyền dưới dạng đối số cho effect function. Tuy nhiên, về mặt khái niệm, đó là những gì họ thể hiện: mọi giá trị được tham chiếu bên trong effect function cũng sẽ xuất hiện trong mảng phụ thuộc. Trong tương lai, một trình biên dịch đủ sịn xò để có thể tạo ra mảng này.

### `useContext` {#usecontext}

```js
const value = useContext(MyContext);
```

Chấp nhận một context object(giá trị được trả về từ `React.createContext`) và trả về context value hiện tại cho context đó. Context hiện tại được xác định bởi `giá trị` props của `<MyContext.Provider>` gần nhất trên component gọi trong cây (tree).

Khi `<MyContext.Provider>` gần nhất cập nhật component, Hook sẽ kích hoạt một render với context `value` được truyền cho `MyContext` provider.

Đừng quên rằng đối số để sử dụng `useContext` phải là *chính context object*:

 * **Correct:** `useContext(MyContext)`
 * **Incorrect:** `useContext(MyContext.Consumer)`
 * **Incorrect:** `useContext(MyContext.Provider)`

Một component gọi `useContext` sẽ luôn render lại khi giá trị context thay đổi. Nếu render lại component *đắt tiền*(expensive), bạn có thể tối ưu hoá nó bằng [sử dụng ghi nhớ(memoization)](https://github.com/facebook/react/issues/15156#issuecomment-474590693).

>Mẹo
>Nếu bạn quen thuộc với context API trước Hook, `useContext(MyContext)` tường đương với `static contextType = MyContext` trong class, hoặc với `<MyContext.Consumer>`.
>
>`useContext(MyContext)` chỉ cho phép bạn đọc context và đăng ký thay đổi đăng ký(subscribe) của nó. Bạn vẫn cần một `<MyContext.Provider>` ở tree để cung cấp giá trị cho context.

## Hooks bổ sung {#additional-hooks}

Các Hook sau đây là các biến thể của Hook cơ bản từ phần trước hoặc chỉ cần cho các trường hợp cụ thể. Không nhấn mạnh về việc học chúng lên trước.

### `useReducer` {#usereducer}

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

Một thay thế cho `useState`. Nó cho phép một reducer loại `(state, action) => newState`, và trả về state hiện tại được ghép nối với một phương thức `dispatch`. (Nếu bạn quen thuộc với Redux, bạn đã biết cách thức hoạt động của nó)

`useReducer` thường được ưu tiên sử dụng state khi có state logic phức tạp liên quan đến nhiều giá trị phụ hoặc khi state tiếp theo phụ thuộc vào state trước đó. `useReducer` cũng cho phép bạn tối ưu hoá hiệu suất cho các component kích hoạt cập nhật sâu vì [bạn có thể `dispath` xuống thay vì callback](/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down).

Đây là ví dụ về count từ phần [`useState`](#usestate) trước đó được viết lại sử dụng reducer:

```js
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter({initialState}) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
    </>
  );
}
```

>Chú ý

>React đảm bảo rằng danh tính `dispatch` function ổn định và giành được sự thay đổi khi render lại. Đây là lý do tại sao nó lại an toàn khi bỏ qua danh sách phụ thuộc `useEffect` hoặc `useCallback`.

#### Chỉ định trạng thái ban đầu {#specifying-the-initial-state}

Có hai cách khác nhau để khởi tạo `useReducer` state. Bạn có thể chọn một trong hai tuỳ thuộc vào trường hợp sử dụng. Cách đơn giản nhất để pass state ban đầu là đối số thứ hai:

```js
  const [state, dispatch] = useReducer(
    reducer,
    {count: initialCount}
  );
```

>Chú thích
>
>React không sử dụng quy ước đối số `state = initialState` phổ biến bở Redux. Giá trị ban đầu đôi khi cần phụ thuộc vào props và do đó được chỉ định từ lệnh gọi Hook thay thế. Nếu bạn cảm thấy bạn đủ giỏi, bạn có thể gọi `useReducer(reducer, undefined, reducer)` để mô phỏng hành vi Redux, nhưng nó không được khuyến khích.

#### Lazy initialization {#lazy-initialization}

Bạn cũng có thể tạo state ban đầu một cách lười biếng(lazy). Để làm điều này, bạn có thể truyền một `init` function làm đối số thử ba. State ban đầu sẽ được đặt thành `init(initialArg)`.

Nó cho phép bạn trích xuất(extract) logic để tính toán state ban đầu bên ngoài reducer. Điều này cũng thuận tiện cho việc đặt lại state sau này để phản hồi(response) một hành động.

```js{1-3,11-12,19,24}
function init(initialCount) {
  return {count: initialCount};
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({type: 'reset', payload: initialCount})}>
        Reset
      </button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
    </>
  );
}
```

#### Bailing out of a dispatch {#bailing-out-of-a-dispatch}

If you return the same value from a Reducer Hook as the current state, React will bail out without rendering the children or firing effects. (React uses the [`Object.is` comparison algorithm](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description).)

Note that React may still need to render that specific component again before bailing out. That shouldn't be a concern because React won't unnecessarily go "deeper" into the tree. If you're doing expensive calculations while rendering, you can optimize them with `useMemo`.

### `useCallback` {#usecallback}

```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

Returns a [memoized](https://en.wikipedia.org/wiki/Memoization) callback.

Pass an inline callback and an array of dependencies. `useCallback` will return a memoized version of the callback that only changes if one of the dependencies has changed. This is useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (e.g. `shouldComponentUpdate`).

`useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.

> Note
>
> The array of dependencies is not passed as arguments to the callback. Conceptually, though, that's what they represent: every value referenced inside the callback should also appear in the dependencies array. In the future, a sufficiently advanced compiler could create this array automatically.
>
> We recommend using the [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) rule as part of our [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) package. It warns when dependencies are specified incorrectly and suggests a fix.

### `useMemo` {#usememo}

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

Returns a [memoized](https://en.wikipedia.org/wiki/Memoization) value.

Pass a "create" function and an array of dependencies. `useMemo` will only recompute the memoized value when one of the dependencies has changed. This optimization helps to avoid expensive calculations on every render.

Remember that the function passed to `useMemo` runs during rendering. Don't do anything there that you wouldn't normally do while rendering. For example, side effects belong in `useEffect`, not `useMemo`.

If no array is provided, a new value will be computed on every render.

**You may rely on `useMemo` as a performance optimization, not as a semantic guarantee.** In the future, React may choose to "forget" some previously memoized values and recalculate them on next render, e.g. to free memory for offscreen components. Write your code so that it still works without `useMemo` — and then add it to optimize performance.

> Note
>
> The array of dependencies is not passed as arguments to the function. Conceptually, though, that's what they represent: every value referenced inside the function should also appear in the dependencies array. In the future, a sufficiently advanced compiler could create this array automatically.
>
> We recommend using the [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) rule as part of our [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) package. It warns when dependencies are specified incorrectly and suggests a fix.

### `useRef` {#useref}

```js
const refContainer = useRef(initialValue);
```

`useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument (`initialValue`). The returned object will persist for the full lifetime of the component.

A common use case is to access a child imperatively:

```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

Essentially, `useRef` is like a "box" that can hold a mutable value in its `.current` property.

You might be familiar with refs primarily as a way to [access the DOM](/docs/refs-and-the-dom.html). If you pass a ref object to React with `<div ref={myRef} />`, React will set its `.current` property to the corresponding DOM node whenever that node changes.

However, `useRef()` is useful for more than the `ref` attribute. It's [handy for keeping any mutable value around](/docs/hooks-faq.html#is-there-something-like-instance-variables) similar to how you'd use instance fields in classes.

This works because `useRef()` creates a plain JavaScript object. The only difference between `useRef()` and creating a `{current: ...}` object yourself is that `useRef` will give you the same ref object on every render.

Keep in mind that `useRef` *doesn't* notify you when its content changes. Mutating the `.current` property doesn't cause a re-render. If you want to run some code when React attaches or detaches a ref to a DOM node, you may want to use a [callback ref](/docs/hooks-faq.html#how-can-i-measure-a-dom-node) instead.


### `useImperativeHandle` {#useimperativehandle}

```js
useImperativeHandle(ref, createHandle, [deps])
```

`useImperativeHandle` customizes the instance value that is exposed to parent components when using `ref`. As always, imperative code using refs should be avoided in most cases. `useImperativeHandle` should be used with `forwardRef`:

```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

In this example, a parent component that renders `<FancyInput ref={fancyInputRef} />` would be able to call `fancyInputRef.current.focus()`.

### `useLayoutEffect` {#uselayouteffect}

The signature is identical to `useEffect`, but it fires synchronously after all DOM mutations. Use this to read layout from the DOM and synchronously re-render. Updates scheduled inside `useLayoutEffect` will be flushed synchronously, before the browser has a chance to paint.

Prefer the standard `useEffect` when possible to avoid blocking visual updates.

> Tip
>
> If you're migrating code from a class component, note `useLayoutEffect` fires in the same phase as `componentDidMount` and `componentDidUpdate`. However, **we recommend starting with `useEffect` first** and only trying `useLayoutEffect` if that causes a problem.
>
>If you use server rendering, keep in mind that *neither* `useLayoutEffect` nor `useEffect` can run until the JavaScript is downloaded. This is why React warns when a server-rendered component contains `useLayoutEffect`. To fix this, either move that logic to `useEffect` (if it isn't necessary for the first render), or delay showing that component until after the client renders (if the HTML looks broken until `useLayoutEffect` runs).
>
>To exclude a component that needs layout effects from the server-rendered HTML, render it conditionally with `showChild && <Child />` and defer showing it with `useEffect(() => { setShowChild(true); }, [])`. This way, the UI doesn't appear broken before hydration.

### `useDebugValue` {#usedebugvalue}

```js
useDebugValue(value)
```

`useDebugValue` can be used to display a label for custom hooks in React DevTools.

For example, consider the `useFriendStatus` custom Hook described in ["Building Your Own Hooks"](/docs/hooks-custom.html):

```js{6-8}
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  // Show a label in DevTools next to this Hook
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
```

> Tip
>
> We don't recommend adding debug values to every custom Hook. It's most valuable for custom Hooks that are part of shared libraries.

#### Defer formatting debug values {#defer-formatting-debug-values}

In some cases formatting a value for display might be an expensive operation. It's also unnecessary unless a Hook is actually inspected.

For this reason `useDebugValue` accepts a formatting function as an optional second parameter. This function is only called if the Hooks are inspected. It receives the debug value as a parameter and should return a formatted display value.

For example a custom Hook that returned a `Date` value could avoid calling the `toDateString` function unnecessarily by passing the following formatter:

```js
useDebugValue(date, date => date.toDateString());
```
