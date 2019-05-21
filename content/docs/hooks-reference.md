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

#### Bailing out of a dispatch(???) {#bailing-out-of-a-dispatch}

Nếu bạn trả về cùng một dữ liệu từ Reducer Hook như state hiện tại, React sẽ đảm bảo không hiển thị các effect con hoặc bắn các effect. (React sử dụng [`thuật toán so sánh Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description).)

Lưu ý rằng React có thể vẫn cần render lại component cụ thể nào đó trước khi bảo lãnh(bailing out). Nếu bạn đang thực hiện các phép tính trong khi render, bạn có thể tối ưu hoá chúng bằng `useMemo`.

### `useCallback` {#usecallback}

```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

Trả về một callback ghi nhớ([memoized](https://en.wikipedia.org/wiki/Memoization) callback)

Vượt qua một lời gọi nội tuyến (inline callback) và một array of dependencies. `useCallback` sẽ trả về một phiên bản ghi nhớ(memoized version) của callback chỉ thay đổi nếu một trong các dependencies thay đổi. Điều này hữu ích khi chuyển các callback đến các component con được tối ưu hoá để ngăn chặn việc render không cần thiết (ví dụ `shouldComponentUpdate`).


`useCallback(fn, deps)` tương đương `useMemo(() => fn, deps)`.

> Chú ý
>
>Mảng phụ thuộc(array of dependencies) không được truyền dưới dạng đối số cho callback, Tuy nhiên, về mặt khái niệmm đó là những gì nó thể hiện: Mọi quá trị được tham chiếu bên trong hàm callback cũng sẽ xuất hiện trong array dependencies. Trong tương lai, một trình biên dịch sịn xò có thể tự động tạo mảng này.
>
Chúng tôi khuyên bạn sử dụng quy tắc [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) là một phần của [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation). Nó cảnh báo khi các dependencies được chỉ định không chính xác và đề nghị sửa chữa.

### `useMemo` {#usememo}

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

Trả về một giá trị ghi nhớ([memoized](https://en.wikipedia.org/wiki/Memoization)).

Chấp nhận một "create" function và một array dependencies. `useMomo` sẽ chỉ tính lại giá trị ghi nhớ khi một trong các dependencies thay đổi. Tối ưu hoá này giúp tránh các tính toán ở mỗi lần render.

Hãy nhớ rằng function được truyền cho `useMemo` chạy trong khi render. Đừng làm điều gì mà bạn sẽ thường làm khi render. Ví dụ:  side effect về `useEffect`, không sử dụng `useMemo`.

Nếu không có mảng nào được cung cấp, một giá trị mới sẽ được tính trên mỗi lần render.

**Bạn có thể dựa vào `useMemo` như một tối ưu hoá hiệu suất, không phải là một đảm bảo ngữ nghĩa.** Trong tương lai, React có thể chọn để quên đi một số giá trị được ghi nhớ trước đó và tính toán lại chúng trong lần render tiếp theo. ví dụ: để giải phóng bộ nhớ cho các component ngoài màn hình. Code của bạn có thể vẫn hoạt động bình thường mà không cần sử dụng `useMemo` - và sau đó thêm nó để tối ưu hoá hiệu suất.

> Chú ý
>
> Mảng phụ thuộc(array of dependencies) không được truyền dưới dạng đối số cho callback, Tuy nhiên, về mặt khái niệmm đó là những gì nó thể hiện: Mọi quá trị được tham chiếu bên trong hàm callback cũng sẽ xuất hiện trong array dependencies. Trong tương lai, một trình biên dịch sịn xò có thể tự động tạo mảng này.
>
> Chúng tôi khuyên bạn sử dụng quy tắc exhaustive-deps là một phần của eslint-plugin-react-hooks. Nó cảnh báo khi các dependencies được chỉ định không chính xác và đề nghị sửa chữa

### `useRef` {#useref}

```js
const refContainer = useRef(initialValue);
```

`useRef` trả về một ref object có thể thay đổi, có thuộc tính `.current` hiện được khởi tạo cho đối số được truyền (`initialValue`). Object trả về sẽ tồn tại trong suất bòng đời của component.

Một trường hợp sử dụng phổ biến là truy cập một child imperatively:

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

Về cơ bản, `useRef` giống như cái hộp, có thể chứa giá trị có thể thay đổi thuộc tính hiện tại (`.current` property) của nó.

Bạn có thể đã quen thuộc với các ref như cách [truy cập DOM](/docs/refs-and-the-dom.html). Nếu bạn di chuyển một đối tưởng ref cho React với `<div ref={myRef} />`, React sẽ đặt thuộc tính hiện tại của nó cho nút DOM tương ứng bất cứ khi nào nút đó thay đổi.

Tuy nhiên, `useRef()` hữu ích hơn nhiều so với thuộc tính `ref`. Nó có sẵn để lưu bất cứ giá trị có thể thay đổi nào tương tự như cách bạn sử dụng các trường đối tượng trong class.

Điều này hoạt động vì `useRef()` tạo một đối tượng Javascript đơn giản.  Sự khác biệt duy nhất giữa `useRef()` và tự tạo một đối tượng `<div ref={myRef} />` là `useRef` sẽ cung cấp cho bạn cùng một object trên mỗi lần render.

Hãy nhớ rằng `useRef` *không* thông báo cho bạn khi nội dung của nó thay đổi. Đột biến(mutating) thuộc tính hiện tại(`.current property) không thể làm render lại. Nếu bạn muốn chạy một số mã khi React đính kèm hoặc tách một ref cho nút DOM, bạn có thể muốn sử dụng [callback ref](/docs/hooks-faq.html#how-can-i-measure-a-dom-node)


### `useImperativeHandle` {#useimperativehandle}

```js
useImperativeHandle(ref, createHandle, [deps])
```

`useImperativeHandle` tuỳ chỉnh giá trị để được hiển thị cho các component cha mẹ khi sử dụng `ref`. Như mọi khi, code bắt buộc sử dụng ref nên tránh trong hầu hết các trường hợp. `useImperativeHandle` nên được sử dụng với `forwardRef`:

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

Trong ví dụ này, một component cha mẹ (parent component) render ra `<FancyInput ref={fancyInputRef} />` sẽ có thể gọi `fancyInputRef.current.focus()`.

### `useLayoutEffect` {#uselayouteffect}

Phần này giống `useEffect`, nhưng nókichs hoạt đồng bộ sau tất cả các đột biến(mutation) DOM. Sử dụng điều này để đọc bố cục từ DOM và đồng bộ việc render lại. Các bản cập nhật được lên lịch trong`useLayoutEffect` sẽ được xoá đồng bộ, trước khi trình duyệt có cơ hội vẽ

Sử dụng `useEffect` khi có thể để tránh việc cập nhật trực quan

> Mẹo
>
>Nếu bạn di chuyển code từ một class component, hãy lưu ý `useLayoutEffect` kích hoạt cùng pha với `componentDidMount` và `componentDidUpdate`. Tuy nhiên, chúng tôi khuyên bạn nên bắt đầu với `useEffect` trước và chỉ sử dụng `useLayoutEffect` nếu điều đó sảy ra sự cố. 
>
>Nếu bạn dùng server rendering, hãy nhớ rằng không sử dụng `useLayoutEffect` hay `useEffect` có thể chạy cho đến khi Javascript được tải xuống. Đây là lý do tại sao React cảnh báo khi một component server-rendered sử dụng `useLayoutEffect`. Để khắc phục điều này, hãy chuyển logic đó sang `useEffect`(nếu nó không cần thiết cho lần render đầu tiên) hoặc trì hoãn hiển thị component đó cho đến khi máy khách render lại( nếu HTML có vẻ bị hỏng cho đến khi `useLayoutEffect chạy)
>
>Để loại trừ một component cần hiệu ứng bố cục khỏi HTML server-rendered, render nó một cách có điều kiện với `showChild && <Child />` và trì hoãn hiển thị nó với `useEffect(() => { setShowChild(true); }, [])`. Bằng cách này, UI không xuất hiện trước hydration.

### `useDebugValue` {#usedebugvalue}

```js
useDebugValue(value)
```

`useDebugValue` có thể được sử dụng để hiển thị nhãn(label) cho custom hook trong React DevTools.

Ví dụ, hãy xem xét custom Hook `useFriendStatus` được mô tả trong ["xây dựng Hook cho riêng bạn"](/docs/hooks-custom.html):

```js
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  // Show a label in DevTools next to this Hook
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
```

> Mẹo
>
> Chúng tôi khuyên bạn nên thêm các giá trị gỡ lỗi cho mỗi custom Hook. Nó có giá trị nhất đối với custom hook là một phần của các thư viện dùng chung.

#### Triển khai định dạng các giá trị gỡ lỗi {#defer-formatting-debug-values}

Trong một số trường hợp, định dạng lại giá trị để hiển thị có thể là một thao tác quan trọng. Nó cũng không cần thiết trừ khi một Hook thực sự được kiểm tra.

Vì lý do này, `useDebugValue` chấn nhận formatting function như một dạng tham số thứ hai tuỳ chọn. Function này chỉ được gọi nếu Hook được kiểm tra. Nó nhận giá trị gỡ lỗi dưới dạng tham số và trả về hiển thị đã được định dạng


Ví dụ: Custom Hook trả về giá trị `Date` có thể tránh gọi function `toDateString` một cách không cần thiết bằng cách chuyển định dạng sau:

```js
useDebugValue(date, date => date.toDateString());
```
