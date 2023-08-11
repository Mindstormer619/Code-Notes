# React Basics
> Created: 2022-03-07 16:12

```tsx
const Component = () => {
	return <h1>This is a component!</h1>;
}

export default Component;
```

We can put HTML directly inside TSX code for React components, and the above HTML will get resolved to

```tsx
React.createElement('h1', {}, 'This is a component!')
```

## JS Expressions
You can evaluate JavaScript expressions inside `{}` in components' JSX return expressions.

```tsx
const Component = () => {
	const name = "Cold";
	return <h1>Listen to song {name}!</h1>
}
```

## Hooks
React provides 3 event hooks for components:
1. When mounted
2. When unmounted
3. When modified

## Props

It's the #1 recommended way to change the output of a React component. Properties can be passed in via HTML-like attributes.

```tsx
// You can specify any interface, or just `any` instead
interface ComponentProps {
	name: string;
}

const Component = (props: ComponentProps) => {
	return <h1>Listen to song {props.name}!</h1>;
}

const ParentComponent = () => {
	return <Component name="Internal Flight" />;
}
```

You can specify the type of the method as a React functional component instead:

```tsx
const Component: React.FC<ComponentProps> = (props) => {
	return <h1>Listen to song {props.name}!</h1>;
}
```

## State

```tsx
import { useState } from 'react';

const [value, setValue] = useState('initialValue');

// Using the value
<h2>{value}</h2>

// updating the value
<h1 onClick={() => setValue('newValue')}>Click me</h1>
```

`useState(v)` returns an array, where the first element is a variable containing the value of the state, and the second element is a function which can be called to update the state.

----

## References
1. Subbu Sessions (The Hitchhiker's Guide to Building Webapps)