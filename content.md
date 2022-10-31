# Frontend Unit Testing

Knowledge Sharing by *@ilumin*

---

## 🤷🏻‍♂️ Why

Why I want to share this?

///

### Testing Frontend is Hard

- it's easy to mess thing up
- it always changes (and very quick)

///

### I experienced

- fixing lot of test code when refactor
- writing too much test code
- code that hard to test

///

### I want

- joy of coding and refactoring
- write less code
- increase effort (and quality)

<!-- 
- so I look back in my experience 
- and research more on unit testing
- 'cause I believe good test bring joy to developer
-->

///

### unit test that

- Prevent regression
- Fast feedback
- Maintainability

---

## 🧐 What I suggested?

---

I suggested

### Avoid tight coupling

<small>(nickname: [avoid testing implementation detail](https://kentcdodds.com/blog/testing-implementation-details))</small>

between test code and implementation code

///

Counter component

```jsx [|2|4-5,8-10|9|5|2|8]
const Counter = () => {
  const [count, setCount] = useState(0)

  const decrease = () => setCount(c => c - 1)
  const increase = () => setCount(c => c + 1)

  return <div>
    <span>Counter: {count}</span>
    <button onClick={increase}>Increse</button>
    <button onClick={decrease}>Decrese</button>
  </div>
}
```

///

```js [1-7|3|4|5|6|3-6]
test('increase value when click increase', () => {
  const container = render(<Counter />)
  const message = container.querySelector('span')
  const button = container.querySelectorAll('button')
  button[0].click()
  expect(input.textContent).toBe('Counter: 1')
})
```

///

```jsx [8-20|11,15,16]
const Counter = () => {
  const [count, setCount] = useState(0)

  const decrease = () => setCount(c => c - 1)
  const increase = () => setCount(c => c + 1)

  return (
    <Paper>
      <Grid container>
        <Grid item xs zeroMinWidth p={2} display="flex" flexDirection="column" justifyContent="center" alignItem="center">
          <Typography role="presentation" aria-label="counter">{count}</Typography>
        </Grid>
        <Grid item>
          <ButtonGroup variant="contained" orientation="vertical" size="small">
            <Button aria-label="increase" onClick={increase}><Icon>add</Icon></Button>
            <Button aria-label="decrease" onClick={decrease}><Icon>remove</Icon></Button>
          </ButtonGroup>
        </Grid>
      </Grid>
    </Paper>
  )
}
```

<iframe src="https://stackblitz.com/edit/react-4blzg3?embed=1&file=demo.tsx&hideDevTools=1&hideExplorer=1&hideNavigation=1&theme=dark&view=preview" />

///

```js [3-4|5-6|8|10]
test('increase value when click increase', () => {
  const { getByRole } = render(<Counter />)
  const increase = getByRole('increase', {name: /increase/i})
  const counter = getByRole('presentation', {name: /counter/i})
  
  userEvent.click(increase)

  expect(counter.textContent).toBe(1)
})
```

///

```jsx [|2|4-5,8-10|9|5|2|8]
const Counter = () => {
  const [count, setCount] = useState(0)

  const decrease = () => setCount(c => c - 1)
  const increase = () => setCount(c => c + 1)

  return <div>
    <span>
      Counter: 
      <span role="presentation" aria-label="counter">{count}</span>
    </span>
    <button aria-label="increase" onClick={increase}>Increse</button>
    <button aria-label="decrease" onClick={decrease}>Decrese</button>
  </div>
}
```

///

### [Query Priority](https://testing-library.com/docs/queries/about/#priority)

1. Queries Accessible for Everyone
   - `getByRole`
   - ...
2. Semantic Quries
3. TestIDs
   - `getByTestId`

---

I suggested

## Don't Use Snapshot Test

///

why?

- tight couple on things that easily change <!-- .element: class="fragment" -->
- snapshot file is hard to read <!-- .element: class="fragment" -->

///

if you are too lazy  
you can use `toMatchInlineSnapshot`

```jsx []
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://example.com">Example Site</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot(⭐️);
});
```

test will update expected text inside the function <!-- .element: class="fragment" -->

///

If you want to verify design, use

### Visual Testing

<small>Storybook also [support](https://storybook.js.org/docs/react/writing-tests/visual-testing)</small>

---

I suggested

## Don't Mock too Much

///

### A long long time ago

There are 2 temples of unit testing  <!-- .element: class="fragment" -->

1. Temple of Jedi (London school or Mockist) <!-- .element: class="fragment" -->
2. Temple of Sith (Detroit school or Classic) <!-- .element: class="fragment" -->

<!-- 
both of them argued about the definition of how to isolate the unit of code
-->

///

### London school

prefer mock on all dependencies

<!-- 
London believe that unit is the code we are about to test
so, mock every dependencies
-->

///

### Detroit school

prefer mock on shared dependencies

<!-- 
Detroit believe that unit is the test itself because we're focusing on test behavior not the code
so, only the shared dependencies should be mock
-->

///

### Dependencies

- shared dependency
- private dependency

![](https://drek4537l1klr.cloudfront.net/khorikov/Figures/02fig03_alt.jpg)

<!-- 
shared dependency: dependency that is shared betweeb test (if it mutate, it effect others)
private dependency: dependency that doesn't share
-->

///

Shared dependency in Frontend

- API
- Cookie

///

### Mock API?

```txt
├─ Component
|  ├─ Redux
|  |  ├─ Saga
|  |  |  ├─ Fetch
|  |  |  |  ├─ API Call
```

///

### MSW

Mock Service Worker

```jsx [|2|3-6|7-9]
test('loads greetings on click', async () => {
  render(<GreetingLoader />)
  const nameInput = screen.getByLabelText(/name/i)
  const loadButton = screen.getByText(/load/i)
  userEvent.type(nameInput, 'Mary')
  userEvent.click(loadButton)
  await waitFor(() =>
    expect(screen.getByLabelText(/greeting/i)).toHaveTextContent('Hello Mary'),
  )
})
```

///

```jsx [|1-8|5-7|6|10-12|14-23|15|16-19|20-22]
import {rest} from 'msw'
import {setupServer} from 'msw/node'

const server = setupServer(
  rest.post('/greeing', (req, res, ctx) => 
    res(ctx.json({data: {greeting: `Hello ${req.body.subject}`}}))
  ),
)

beforeAll(() => server.listen())
afterAll(() => server.close())
afterEach(() => server.resetHandlers())

test('loads greetings on click', async () => {
  render(<GreetingLoader />)
  const nameInput = screen.getByLabelText(/name/i)
  const loadButton = screen.getByText(/load/i)
  userEvent.type(nameInput, 'Mary')
  userEvent.click(loadButton)
  await waitFor(() =>
    expect(screen.getByLabelText(/greeting/i)).toHaveTextContent('Hello Mary'),
  )
})
```

---

## 🙋 Question?

---

## ⏳ What's Next?

- [feedback](https://forms.gle/8NjQhZNQWvSB3nvJ6)
- unit testing with Storybook (unit and integrate level)

---

## 📚 Resources

- [Manning, Micro Frontend in Action](https://www.manning.com/books/micro-frontends-in-action)
- [Kent C. Dodds, Avoid the Test User](https://kentcdodds.com/blog/avoid-the-test-user)
- [Kent C. Dodds, Testing Implementation Details](https://kentcdodds.com/blog/testing-implementation-details)
