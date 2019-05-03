---
id: js-custom
title: Building Custom UI
sidebar_label: Custom UI
---

Displaying your data in a table might work for many use-cases. However, depending on your plugin and data it might make sense to customize the way your data is visualized. Flipper uses React to render the plugins and provides a variety of ready-to-use UI components that can be used to build custom plugin UIs.

## Replacing the table

For our sea mammals app, we might not only want to see them listed as image URLs in a table but render the actual images in nice little cards. When selecting one of the cards we still want to display all details in the sidebar.
![Custom cards UI for our sea mammals plugin](/docs/assets/js-custom.png)

Currently, the default export in our `index.js` is from `createTablePlugin`. Now we are going to replace this with a custom React component extending `FlipperPlugin`.

```js
export default class SeaMammals extends FlipperPlugin {
  static Container = styled(FlexRow)({
    backgroundColor: colors.macOSTitleBarBackgroundBlur,
    flexWrap: 'wrap',
    alignItems: 'flex-start',
    alignContent: 'flex-start',
    flexGrow: 1,
  });

  render() {
    return (
      <SeaMammals.Container>
        Hello custom plugin!
      </SeaMammals.Container>
    );
  }
}
```

You can see how we are styling our components using [emotion](https://emotion.sh/). To learn more about this, make sure to read our guide on [styling components](extending/styling-components.md).

## Adding data handling

The plugin is quite useless when we don't display any actual data. We are adding two static properties to our plugin class for data handling. `defaultPersistedState` defines the default state before we received any data. In `persistedStateReducer` we define how new data is merged with the existing data.

For the default state we define an empty array because we don't have any data, yet. When receiving data, we simply add it to the existing array. Learn more about [persistedState](extending/js-plugin-api.md#persistedstate) in our guide.

```js
static defaultPersistedState = {
  data: [],
};

static persistedStateReducer = (
  persistedState: PersistedState,
  method: string,
  payload: Row,
) => {
  if (method === 'newRow') {
    return {
      data: [...persistedState.data, payload],
    };
  }
  return persistedState;
};
```

Note: The method name `newRow` is still the same that we defined on the native side.

## Rendering the data

Now we can access the data from `this.props.persistedState.data` and render it. So let's update our `render` method using a custom `Card` component, which we will implement in a bit.

```js
render() {
  const {selectedIndex} = this.state;
  const {data} = this.props.persistedState;

  return (
    <SeaMammals.Container>
      {data.map((row, i) => <Card
        {...row}
        key={i}
        onSelect={() => this.setState({selectedIndex: i})}
        selected={i === selectedIndex}
      />)}
    </SeaMammals.Container>
  );
}
```

## Adding the sidebar

When clicking on a Card, we want to show all details in the sidebar as we did with the table before. We are using React's state to store the selected index in our array. Flipper provides a `DetailSidebar` component which we can use to add information to the sidebar. It doesn't matter where this component is placed as long as it is returned somewhere in our `render` method. The `renderSidebar` method returning the sidebar's content is still the same we used with `createTablePlugin`.

```js
<DetailSidebar>
  {this.state.selectedIndex > -1 &&
    renderSidebar(this.props.persistedState.data[selectedIndex])}
</DetailSidebar>
```


## Creating a custom component

The `Card` component is responsible for rendering the actual image and title. This is not very specific to Flipper, but is using plain React. Note the usage of `styled` to adjust the style of existing UI components and `colors` which provides a library of colors used throughout the app.

```js
class Card extends React.Component<{
  ...Row,
  onSelect: () => void,
  selected: boolean,
}> {
  static Container = styled(FlexColumn)(props => ({
    margin: 10,
    borderRadius: 5,
    border: '2px solid black',
    backgroundColor: colors.white,
    borderColor: props.selected
      ? colors.macOSTitleBarIconSelected
      : colors.white,
    padding: 0,
    width: 150,
    overflow: 'hidden',
    boxShadow: '1px 1px 4px rgba(0,0,0,0.1)',
    cursor: 'pointer',
  }));

  static Image = styled('div')({
    backgroundSize: 'cover',
    width: '100%',
    paddingTop: '100%',
  });

  static Title = styled(Text)({
    fontSize: 14,
    fontWeight: 'bold',
    padding: '10px 5px',
    overflow: 'hidden',
    textOverflow: 'ellipsis',
    whiteSpace: 'nowrap',
  });

  render() {
    return (
      <Card.Container
        onClick={this.props.onSelect}
        selected={this.props.selected}>
        <Card.Image style={{backgroundImage: `url(${this.props.url || ''})`}} />
        <Card.Title>{this.props.title}</Card.Title>
      </Card.Container>
    );
  }
}
```