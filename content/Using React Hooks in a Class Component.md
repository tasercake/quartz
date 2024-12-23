[Hooks](https://reactjs.org/docs/hooks-overview.html) have grown into a fundamental part of building React apps since their introduction in late 2018.

Aside from the fundamental React hooks like [`useState`](https://reactjs.org/docs/hooks-state.html) and [`useEffect`](https://reactjs.org/docs/hooks-effect.html), the speed and convenience that hooks like [useSWR](https://swr.vercel.app/), [useQueryParams](https://github.com/pbeshai/use-query-params#usequeryparams), and [useLocalStorage](https://usehooks.com/useLocalStorage/) bring to app development is hard to give up once you've had a taste.

If you're working on an older React project that still uses some (or many) [class-based components](https://www.w3schools.com/react/react_class.asp), chances are you've skipped over using a neat third-party hook because it was too much of a hassle to use it with class components.

To use that fancy new hook in your class component, you have a few options:

# 🐸 Rewrite your class component as a function component

If your component is small enough, it might be wise to simply refactor it as a functional component and save yourself a good deal of pain down the road.

[Here's a great guide to doing just that](https://medium.com/@benjaminmorali4/refactoring-react-class-components-to-typescript-functional-components-with-hooks-a4f42b2bd7b5)

But of course, this doesn't work for everyone.

# 🎁 Wrap a hook in a [Higher-Order Component](https://reactjs.org/docs/higher-order-components.html)

If all you need is a one-off solution and it isn't worth refactoring your entire component, you can create a higher-order component (HOC) wrapper for the specific hook you want to use.

> ⚠️ This approach involves creating a separate HOC wrapper for each hook that you want to use in a class component. If this works for you, great! Otherwise, the next method might be a better alternative.

This will work differently depending on whether your hook expects any parameters.

## Hook without parameters

Consider the following hook that doesn't expect any parameters:

```ts
// The hook's return signature
interface IMonkey {
  monkey: string
  toggle: () => void
}

// Hook that takes no parameters
const useMonkey = (): IMonkey => {
  const [monkey, setMonkey] = useState(false)
  return {
    monkey: monkeySee ? "🐵" : "🙈",
    toggle: () => setMonkey(!monkey) 
  }
}
```

You can create a Higher-Order Component wrapper around the hook that looks something like:

```ts
const withMonkey = <P extends IMonkey>(
  Component: React.ComponentType<P>
) => {
  const Wrapper: React.FunctionComponent<
    Subtract<P, IMonkey>
  > = (props) => {
    const { monkey, toggle } = useMonkey()
    return <Component
      {...(props as P)}
      monkey={monkey}
      toggle={toggle}
    />
  }
  return Wrapper
}
```

> Note that the [`Subtract`](https://www.npmjs.com/package/utility-types#subtractt-t1) type used here comes from the excellent [`utility-types`](https://www.npmjs.com/package/utility-types) package.

And then use it inside any of your class components:

```ts
// Tell TypeScript that your component's props include the values injected by your HOC
interface Props extends IMonkey {
}

// Your class component
class Monkey extends React.Component<Props> {
  render() {
    return (
      <>
        <span>{this.props.monkey}</span>
        <button onClick={this.props.toggle}>Toggle</button>
      </>
    )
  }
}

// Wrap your component with the HOC
export withMonkey(Monkey)
```

## Hook with parameters

If your hook does require some parameters, creating the wrapper is a more involved but still perfectly doable:

```ts
// Parameters that the hook expects
interface HookParams {
  initialState?: boolean
}

// The hook's return signature
interface IMonkey {
  monkey: string
  toggle: () => void
}

// Hook with parameters
const useMonkey = (params: HookParams): IMonkey => {
  const { initialState } = params
  const [monkey, setMonkey] = useState(initialState)
  // ...
}

// HOC that accepts parameters for the hook
export const withMonkey = (params?: HookParams) => {
  return <P extends IMonkey>(Component: React.ComponentType<P>) => {
    const Wrapper: React.FC<Subtract<P, IMonkey>> = (componentProps) => {
      const {monkey, toggle} = useMonkey (params)
      return (
        <Component{...generationAPIHookResult} {...(componentProps as P)}/>  
      )
    }
    return Wrapper
  }
}

interface Props extends IMonkey{
}
class Monkey extends React.Component<Props> {
  // ...
}

// Wrap your component in the HOC, while passing parameters to the hook inside.
export withMonkey({initialState: false})(Monkey)
```

# ✨ Turn _any_ hook into a Higher-Order Component

We can go a step further by generalising the HOC wrapper approach to **_any_ hook that takes no parameters**.

> If your hook takes any parameters, this code won't play well with TypeScript. Check out [`hocify`](https://github.com/ricokahler/hocify) if you don't care too much about satisfying the type checker.

```ts
type HookWithoutParams<R> = () => R    
type PropInjectorHOC<T extends object> = <InjectedProps extends T>(  
  Component: React.ComponentType<InjectedProps>  
) => React.FunctionComponent<Subtract<InjectedProps, T>>

export const makePropInjectorHOCFromHook = <HookResult extends object>(
  hook: HookWithoutParams<HookResult>
): PropInjectorHOC<HookResult> => {
  return <Props extends HookResult>(Component: React.ComponentType<Props>) => {
    const WrapperComponent: React.FunctionComponent<
      Subtract<Props, HookResult>
    > = (props) => {
      const hookResult = hook()
      return <Component {...hookResult} {...(props as Props)} />
    }
    WrapperComponent.displayName = `withInjectedProps(${
      Component.displayName || Component.name
    })`
    return WrapperComponent
  }
}
```

You can use this function to create a HOC, and then use that HOC to wrap your component:

```ts
// Hook that takes no parameters
const useMonkey = (): IMonkey => {
  // ...
}

// Create a HOC from the hook
const withMonkey = makePropInjectorHOCFromHook(useMonkey)

// Wrap your class component with the HOC
interface Props extends IMonkey{
}
class Monkey extends React.Component<Props> {
  // ...
}
export withMonkey(Monkey)
```

# 💡 Next steps

The generalized approach works pretty well for hooks that don't have any parameters, but extending it to hooks with parameters turned out to be a bigger challenge than I had the time for.

It's easy enough to implement in JavaScript, but typing the function - generics and all - is a lot trickier.

# Further Reading

[React Higher-Order Components in TypeScript](https://medium.com/@jrwebdev/react-higher-order-component-patterns-in-typescript-42278f7590fb): This article does a great job of explaining the various kinds of HOCs and the TypeScripts subtleties that go along with them.

[hocify](https://github.com/ricokahler/hocify) if you just want to use hooks in your class components without worrying too much about perfect TypeScript compatibility.