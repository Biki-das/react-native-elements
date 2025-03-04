---
id: customization
title: Customization
---

Congrats! You've installed React Native Elements and your immediate question
goes something like this:

> So umm, how do I change how it looks?

Great question! A UI Kit wouldn't be that useful if the apps everyone built
looked the same right? For this case React Native Elements provide a number of
props on each component to enable you to style them how you want.

## Component Styles

_Every_ component from React Native Elements has a container around it. The
container is just a traditional `<View />` from react native that has some
styling on it. This default styling prevents components from colliding with each
other. If you want to change how two components react to each on the screen your
first stop should be the `containerStyle` prop.

Similar to `containerStyle`, components may provide their custom style props
like `buttonStyle`, `titleStyle` etc. Always refer to the documentation for the
component to find out which style props it provides.

## Theming

While component styles are great for single use, you may want to have the same
styling for every instance of a component. For example, you may want all your
buttons to be blue or have the same font. Here are some ways to reuse styles
with React Native Elements.

### Using Composition

With this approach, we create one component with the styles we want and use that
instead of the built-in component.

```jsx
import React from 'react';
import { Button } from '@react-native-elements/themed';

const RaisedButton = (props) => <Button raised {...props} />;

// Your App
const App = () => {
  return <RaisedButton title="Yea" />;
};
```

If we want to use a button that's raised in our app, we can use `RaisedButton`
instead of using `Button`. This component still accepts all the props from the
normal `Button` just that it has the `raised` prop set by default.

### Using ThemeProvider

The previous solution works great for only one component, but imagine having to
do this for every component you want custom styles for. That could get a bit
tedious to manage. Thankfully, there's a better way to do this. React Native
Elements ships with a 3 utilities for large-scale theming.

Firstly you'll want to set up your `ThemeProvider`.

```jsx
import {
  ThemeProvider,
  Button,
  createTheme,
} from '@react-native-elements/themed';

const theme = createTheme({
  Button: {
    raised: true,
  },
});

// Your App
const App = () => {
  return (
    <ThemeProvider theme={theme}>
      <Button title="My Button" />
      <Button title="My 2nd Button" />
    </ThemeProvider>
  );
};
```

The example above achieves the same goals as the first example — apply the same
styles to multiple instances of `Button` in the app. However this example
applies the `raised` prop to every instance of `Button` inside the component
tree under `ThemeProvider`. Both of these buttons will have the `raised` prop
set to true.

This is extremely convenient and is made possible through
[React's Context API](https://reactjs.org/docs/context.html).

---

### Light and dark mode

React Native Elements also provides a preset dark mode palette to get you started with using dark mode in your app.
Use the prop `mode` in `createTheme` to set the default dark theme. You may want to set this by using a button,
or by using the user's configured settings

```jsx
import {
  ThemeProvider,
  Button,
  createTheme,
} from '@react-native-elements/themed';

const myTheme = createTheme({
  colors: {
    primary: '#f2f2f2',
  },
  darkColors: {
    primary: '#121212',
  },
  mode: 'dark',
});

// Your App
const App = () => {
  return (
    <ThemeProvider theme={myTheme}>
      <Button title="My Button" />
    </ThemeProvider>
  );
};
```

But how to switch modes?

```jsx
import { useTheme } from '@react-native-elements/themed';

const App = () => {
  const { updateTheme } = useTheme();

  const switchToDarkMode = () => {
    updateTheme({
      mode: 'dark',
    });
  };

  const toggleTheme = () => {
    updateTheme((theme) => ({
      mode: theme.mode === 'light' ? 'dark' : 'light',
    }));
  };

  return (
    <>
      <Button title="Toggle Theme" onPress={toggleTheme} />
      <Button title="Dark" onPress={switchToDarkMode} />
    </>
  );
};
```

### TypeScript Definitions (extending the default theme)

TypeScript definitions for your theme can be extended by using TypeScript's [declaration merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html) feature. First you need to create a declaration file called `react-native-elements.d.ts` and then declare the module `react-native-elements` and 're-export' the types that you want to extend.

i.e. below we add a custom `p1Style` to the `Text` theme object and we add a bunch of colors to the `colors` object.

```typescript
// react-native-elements.d.ts

export * from 'patch/to/node_modules/react-native-elements';

type RecursivePartial<T> = { [P in keyof T]?: RecursivePartial<T[P]> };

declare module 'react-native-elements' {
  export interface TextProps {
    p1Style: StyleProp<TextStyle>;
  }

  export interface Colors {
    background: string;
    border: string;
    text: string;
    altText: string;
    danger: string;
  }

  export interface FullTheme {
    colors: RecursivePartial<Colors>;
    Text: Partial<TextProps>;
  }
}
```

---

### Order of Styles

What happens now if we want a `Button` that isn't raised? To do that we have to understand the order in which styles are applied.

> Internal > Theme > External

#### Internal

Internal components styles are the styles which are defined in the component
file. These are applied first.

#### Theme

Theme styles are the values that are set by the ThemeProvider If present, these
are applied second.

```jsx
import {
  ThemeProvider,
  Button,
  createTheme,
} from '@react-native-elements/themed';

const theme = createTheme({
  Button: {
    titleStyle: {
      color: 'red',
    },
  },
});

const App = () => {
  return (
    <ThemeProvider theme={theme}>
      <Button title="My Button" />
    </ThemeProvider>
  );
};
```

This will override the white color for the title set in the component's style.

#### External

External styles are the styles which are set through the component props. These
are applied last and have the highest precedence.

```jsx
import { ThemeProvider, Button } from '@react-native-elements/themed';

const theme = {
  Button: {
    titleStyle: {
      color: 'red',
    },
  },
};

const App = () => {
  return (
    <ThemeProvider theme={theme}>
      <Button title="My Button" titleStyle={{ color: 'pink' }} />
    </ThemeProvider>
  );
};
```

This will override both the white color for the title set in the component's
style as well as the red color set in the theme.

> Remember if you want to override the values set in the theme you can always
> use component props.

Note: To theme subcomponents such as `ListItem.Title`, in your theme remove the dot and list them as "ListItemTitle"

---

### The Theme Object

By default, the theme object looks like this. You can add whatever values you
want to the theme, and they will be merged with the default. By default the
platform colors aren't used anywhere. These native colors are added for
your convenience.

```tsx
interface theme {
  colors: {
    primary;
    secondary;
    white;
    black;
    grey0;
    grey1;
    grey2;
    grey3;
    grey4;
    grey5;
    greyOutline;
    searchBg;
    success;
    error;
    warning;
    divider;
    platform: {
      ios: {
        primary;
        secondary;
        grey;
        searchBg;
        success;
        error;
        warning;
      };
      android: {
        // Same as ios
      };
      web: {
        // Same as ios
      };
    };
  };
}
```

Setting styles in the theme is as simple as using the name of the component, as
a key and the props you want to change as the value.

```jsx
import { ThemeProvider ,createTheme} from '@react-native-elements/themed';

const theme = createTheme({
  Avatar: {
    rounded: true,
  },
  Badge: {
    textStyle: { fontSize: 30 },
  },
});

...

<ThemeProvider theme={theme}>
```

---

### Using the theme in your own components

You may want to make use of the theming utilities in your own components. For this you can use the withTheme HOC exported from this library. It adds three props to the component it wraps - theme, updateTheme and replaceTheme.

```tsx title='MyComponent.tsx'
import { Button, createTheme } from '@react-native-elements/themed';

type MyCustomComponentProps = {
  title: string;
  titleStyle: StyleProps<TextStyle>;
};

export const MyCustomComponent = withTheme<MyCustomComponentProps>((props) => {
  // Access theme from props
  const { theme, updateTheme, replaceTheme } = props;
  // ...
});

declare module 'react-native-elements' {
  export interface FullTheme {
    MyCustomComponent: Partial<MyCustomComponentProps>;
  }
}
```

```tsx title='App.tsx'
import { ThemeProvider, createTheme } from '@react-native-elements/themed';

const myTheme = createTheme({
  MyCustomComponent: {
    titleStyle: {
      color: 'red',
    },
  },
});

const App = () => {
  return (
    <ThemeProvider theme={myTheme}>
      <MyCustomComponent title="My Component" />
    </ThemeProvider>
  );
};
```

The updateTheme function merges the theme passed in with the current theme.

```tsx
updateTheme({
  MyCustomComponent: {
    titleStyle: {
      color: 'blue',
    },
  },
});
```

The `replaceTheme` function merges the theme passed in with the default theme.

Don't want to wrap your components? You can use the `ThemeConsumer` component
which uses render props!

```jsx
import React from 'react';
import { Text } from 'react-native';
import { ThemeConsumer } from '@react-native-elements/themed';

const MyComponent = () => (
  <ThemeConsumer>
    {({ theme }) => (
      <Text style={{ color: theme.colors.primary }}>Yo!</Text>;
    )}
  </ThemeConsumer>
)
```

You can also use `useTheme()` if you use hooks.

```jsx
import React from 'react';
import { Text } from 'react-native';
import { useTheme } from '@react-native-elements/themed';

const MyComponent = () => {
  const { theme } = useTheme();

  return (
    <View style={styles.container}>
      <Text style={{ color: theme.colors.primary }}>Yo!</Text>
    </View>
  );
};
```

If you want to keep your styles outside the component use `makeStyles()` (hook generator) to reference the `theme` and component props (optional param).

```jsx
import React from 'react';
import { Text } from 'react-native';
import { makeStyles } from '@react-native-elements/themed';

type Params = {
  fullWidth?: boolean,
};

const MyComponent = (props: Props) => {
  const styles = useStyles(props);

  return (
    <View style={styles.container}>
      <Text style={{ color: theme.colors.primary }}>Yo!</Text>
    </View>
  );
};

const useStyles = makeStyles((theme, props: Props) => ({
  container: {
    background: theme.colors.white,
    width: props.fullWidth ? '100%' : 'auto',
  },
  text: {
    color: theme.colors.primary,
  },
}));
```

---

### Using the respective platform's native colors

You may want to style your app using the native color palette. You can do this
using the `colors` object and the `Platform` API.

```jsx
import { Platform } from 'react-native';
import { Button, colors, ThemeProvider } from '@react-native-elements/themed';

const theme = {
  colors: {
    ...Platform.select({
      default: colors.platform.android,
      ios: colors.platform.ios,
    }),
  },
};

const App = () => {
  return (
    <ThemeProvider theme={theme}>
      // This button's color will now be the default iOS / Android blue.
      <Button title="My Button" />
    </ThemeProvider>
  );
};
```

---

### Common Pitfalls

This section outlines some common pitfalls when using Theming.

#### My local styles aren't working with the theme

It's important to understand that the `ThemeProvider` works by merging your local(external) styles with those set on the theme.
This means that in both cases **the type of these styles must be the same**.

##### Example 1

```jsx
const theme = {
  Button: {
    containerStyle: {
      marginTop: 10;
    }
  }
}

<Button
  containerStyle={{ backgroundColor: 'blue' }}
/>
```

> ✅ Works
>
> In both cases the style is an `object`

<br />

##### Example 2

```jsx
const theme = {
  Button: {
    containerStyle: [
      {
        marginTop: 10;
      }
    ]
  }
}

<Button containerStyle={[{ backgroundColor: 'blue' }]} />
```

> ✅ Works
>
> In both cases the style is an `array`

<br />

##### Example 3

```jsx
const theme = {
  Button: {
    containerStyle: {
      marginTop: 10;
    }
  }
}

<Button containerStyle={[{ backgroundColor: 'blue' }]} />
```

> 🚫 Doesn't work
>
> In one case the style is an `object` and another the style is an `array`
