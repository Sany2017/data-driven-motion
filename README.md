# data-driven-motion

[![npm version](https://badge.fury.io/js/data-driven-motion.svg)](https://badge.fury.io/js/data-driven-motion)
[![Build Status](https://travis-ci.org/tkh44/data-driven-motion.svg?branch=master)](https://travis-ci.org/tkh44/data-driven-motion)
[![codecov](https://codecov.io/gh/tkh44/data-driven-motion/branch/master/graph/badge.svg)](https://codecov.io/gh/tkh44/data-driven-motion)
[![Standard - JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://img.shields.io/badge/code_style-standard-brightgreen.svg)



Easily animate your data in react

This is a small wrapper around [react-motion](https://github.com/chenglou/react-motion) with the intention of simplifying the api for my most common use case.


```bash
npm install -S data-driven-motion
```

## Demos
* [List demo](https://jsfiddle.net/tkh44/vmkjuam4/2/)
* [List demo with multiple layers](https://jsfiddle.net/tkh44/vmkjuam4)
* [Trippy demo](https://jsfiddle.net/tkh44/2j99yb5L/)

## Motion

```jsx
<Motion
  data={[]}
  component={<ul style={{ padding: 8 }} />}
  render={(key, data, style) => <li key={key} style={style}>{data.name}</li>}
  getKey={data => data.name}
  onComponentMount={data => ({ top: data.top, left: data.left })}
  onRender={(data, i, spring) => ({ top: spring(data.top), left: spring(data.left) })}
  onRemount={({ data }) => ({ top: data.top - 32, left: data.left - 32 })}
  onUnmount={({ data }, spring) => ({ top: spring(data.top + 32), left: spring(data.left + 32) })}
/>
```

### Props

#### component: `PropTypes.element`
wrapper element 
ex: `<div/>`


#### data: `PropTypes.array`
array of data

**It does not matter what `data` contains, as long as it is an array**

#### render: `PropTypes.oneOfType([PropTypes.func, PropTypes.arrayOf(PropTypes.func)])`
`(key, data, style) => ReactElement`

called on render for each item in `data` with the `key` from `getKey`, `data` from `data[index]`, and
`style`, the result of: `onComponentMount`, `onRender`, `onRemount`, or `onUnmount`

If an array is provided it will render the resulting elements in order.
This is useful for creating animated layers.


#### getKey: `PropTypes.func`
`(data, i) => string`

help identify which items have been changed, added, or are removed
https://facebook.github.io/react/docs/lists-and-keys.html


#### onComponentMount: `PropTypes.func`
`(data, i) => Style object`

`data === props.data[i]`

called when `component` mounts

__do not wrap values in springs__


#### onRender: `PropTypes.func`
`(data, i, spring) => Style object`

`data === props.data[i]`

called when `props.component` mounts

__ok to wrap values in springs__


#### onRemount: `PropTypes.func`

`({ key, data, style }) => Style object`

*Notice the argument is wrapped in an object*

The argument is the computed config from `onRender` 

__do not wrap values in springs__


#### onUnmount: `PropTypes.func`

`({ key, data, style }) => Style object`

*Notice the argument is wrapped in an object*

The argument is the computed config from `onRender`. 

__ok to wrap values in springs__


### Single Element Entry/Exit
```jsx
<Motion
  data={this.state.show ? [this.props] : []}
  component={<div style={{ padding: 8 }} />}
  render={(key, data, style) => <div key={key} style={style} {...data} />}
  getKey={(data, i) => i}
  onComponentMount={()) => ({ opacity: 0 })}
  onRender={(data, i, spring) => ({ opacity: spring(1) })}
  onRemount={() => ({ opacity: 0 })}
  onUnmount={(config, spring) => ({ opacity: spring(0) })}
/>
```

### List Example
```javascript
import { createElement as h, Component } from 'react'
import { Motion } from 'data-driven-motion'

export default class ListWrapper extends Component {
  render () {
    return h(Motion, {
      data: this.props.data,
      component: <ul />,
      getKey: data => data.name,
      onComponentMount: data => {
        // no springs
        return {
          top: data.top,
          left: data.left,
          opacity: data.top > 500 ? 0 : 1,
          translateY: data.offsetY
        }
      },
      onRender: (data, i) => {
        // springs
        return {
          top: spring(i * data.top),
          left: spring(data.left),
          opacity: data.top > 500 ? 0 : 1, // you don't have to use them,
          translateY: spring(data.offsetY)
        }
      },
      onRemount: ({key, data, style}) => {
        // no springs
        // key, data, style come from the config generated by `onRender`.
        // When the piece of data is remounted this function is called to override the initial position of the rendered element
        // We are creating a starting point to our animation here
        // *from* onRemount styles *to* onRender styles
        // be careful using values from `style` they will be `spring` objects
        return {
          top: data.top,
          left: data.left,
          opacity: data.top > 500 ? 0 : 1,
          translateY: data.offsetY
        }
      },
      onUnmount: ({key, data, style}, i, spring) => {
        // springs
        // Where to animate *to* when removing this data item
        return {
          top: spring(data.top),
          left: spring(data.left),
          opacity: data.top > 500 ? 0 : 1, // you don't have to use them,
          translateY: spring(data.offsetY)
        }
      },
      render: (key, data, style) => {
        // called for each data item with its key, data and style
        return (
          <li
            key={key}
            style={{
              position: 'absolute',
              top: style.top,
              left: style.left,
              opacity: style.opacity,
              transform: `translateY(${style.translateY})`
            }}
            onClick={this.handleClick.bind(this, key, data, style)}
          >
            {data.name} | { data.username}
          </li>
        )
      }
    })
  }
}
```

### React Router v4 AnimatedSwitch Example

_forked from React Router site basic demo_
```jsx
import React from 'react'
import { BrowserRouter as Router, Route, Link, Redirect, matchPath } from 'react-router-dom'
import { Motion } from 'data-driven-motion'

const WOBBLY_SPRING = { stiffness: 200, damping: 15, precision: 0.1 }

const AnimationExample = () => (
  <Router>
    <div>
      <ul>
        <li><Link to='/'>Home</Link></li>
        <li><Link to='/about'>About</Link></li>
        <li><Link to='/topics'>Topics</Link></li>
      </ul>

      <hr />
      <AnimatedSwitch style={{ position: 'relative' }}>
        <AnimatedRoute exact path='/' component={Home} />
        <AnimatedRoute path='/about' component={About} />
        <AnimatedRoute path='/topics' component={Topics} />
      </AnimatedSwitch>
    </div>
  </Router>
)

const Home = ({ style }) => (
  <div style={style}>
    <h2>Home</h2>
    <p>
      All of these examples can be copy pasted into an app created with create-react-app. Just paste the code into
      src/App.js of your project.
    </p>
  </div>
)

const About = ({ style }) => (
  <div style={style}>
    <h2>About</h2>
    <p>
      Components are the heart of React's powerful, declarative programming model. React Router is a collection of
      navigational components that compose naturally with your application. Whether you want to have bookmarkable URLs
      for your web app or a composable way to navigate in React Native, React Router works wherever React is rendering.
    </p>
  </div>
)

const Topics = ({ match, style }) => (
  <div style={{ ...style, height: '100%' }}>
    <h2>Topics</h2>
    <ul style={{ padding: 0, listStyle: 'none' }}>
      <li><Link to={`${match.url}/rendering`}>Rendering with React</Link></li>
      <li><Link to={`${match.url}/components`}>Components</Link></li>
      <li><Link to={`${match.url}/props-v-state`}>Props v. State</Link></li>
    </ul>
    <AnimatedSwitch style={{ position: 'relative' }}>
      <AnimatedRoute
        path={`${match.url}/:topicId`}
        component={Topic}
        getKey={({ match, location }) => {
          return match.url + match.params.topicId
        }}
      />
      <AnimatedRoute exact path={match.url} component={SelectTopic} />
    </AnimatedSwitch>
  </div>
)

const Topic = ({ match, location, style }) => {
  return (
    <div style={style}>
      <h3>{match.params.topicId}</h3>
    </div>
  )
}

const SelectTopic = ({ style }) => <h3 style={style}>Please select a topic.</h3>

class AnimatedSwitch extends React.Component {
  render () {
    const { children, style } = this.props
    const location = this.props.location || this.context.route.location
    let match, child
    React.Children.forEach(children, element => {
      if (match == null) {
        child = element
        match = matchPath(location.pathname, element.props)
      }
    })

    return (
      <Motion
        data={match ? [{ location, match, child }] : []}
        component={<div style={style} />}
        render={(key, data, style) => {
          return React.cloneElement(data.child, {
            key,
            location: data.location,
            computedMatch: data.match,
            style: {
              transform: `translate3d(0, ${style.y}%, 0)`,
              opacity: style.o
            }
          })
        }}
        getKey={({ child, location, match }) => {
          return child.props.getKey // param values used when generating keys
            ? child.props.getKey({ location, match })
            : child.props.path || child.props.from
        }}
        onComponentMount={data => ({ y: 50, o: 0.75 })}
        onRender={(data, i, spring) => ({
          y: spring(0, WOBBLY_SPRING),
          o: spring(1)
        })}
        onRemount={({ data: { child } }) => ({ y: 5, o: 0 })}
        onUnmount={({ data: { child } }, spring) => ({
          y: spring(20, WOBBLY_SPRING),
          o: spring(0)
        })}
      />
    )
  }
}

AnimatedSwitch.contextTypes = {
  route: React.PropTypes.object.isRequired
}

const AnimatedRoute = ({ component: Component, style, getKey, ...rest }) => (
  <Route
    {...rest}
    render={props => (
      <Component
        {...props}
        style={{ position: 'absolute', left: 0, top: 0, ...props.style, ...style }}
      />
    )}
  />
)

export default AnimationExample
```
