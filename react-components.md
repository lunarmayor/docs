##React Components

*the purpose of this guide is to distill our current mental model of how and why particular react components should be written. It is a living document. It should be questioned, challenged, and refined as our collective knowledge grows.*

###Core priciples overview
1. **components should cover a single responsibility**  
 
 ideally, components should only do one thing and no other component should reimplement that same thing. If a component grows too large, it should be broken down into smaller components or leverage composition
2. **components should be data agnostic**  
  
  components should only know what is neccissary for them to propery render and should only be aware of the state that is internal to that particular component.
  
  an image, for example, does not need to know how to fetch a user or even know that is displaying a user's profile image to properly render itself.
  
3. **components should be flexible**  
components should utilize props to make themselves flexible and usuable accross a number of different use cases. components should also avoid implementing subcomponents that are non essential to function of the component.

 for example, a play button and a stop button do not need to be separate components. they can specify different onClick handlers and pass in different icons as their child  
 
 
-


###Priciples in action
let's take a simple feature and apply the three principles. we'll take in in three steps so you can the effects of each. 

the feature we're tasked with building is a cat image that, when clicked, toggles the scales up and down by 10%.

seems simple enough. let's implement it in the worst way first.     

```js
// example of a poorly written component
...

let cat = {
  imageUrl: 'http://avurls.com/23443',
  ...
};

class ScaleableCatImage extends Component {
  constructor(props) {
    super(props);
    
    this.state = {
      cat: null,
      isScaledUp: false,
    };
  }
  
  componentDidMount() {
    // fake fetch data
    this.setState({ cat });
  }
  
  toggleScale() {
    this.setState({
      isScaledUp: !this.state.isScaledUp,
    });
  } 
  
  render() {
    return (
      <div style={styles.imageContainer}>
        { this.state.cat ?
          <Image
            style={[
              styles.image, 
              this.state.isScaledUp && styles.scaledUp 
            ]}
            source={{uri: this.state.cat.imageUrl}} 
            onPress={this.toggleScale.bind(this)}
          />
        : 
          <LoadingIndicator />
        }
      </div>
    )
  }  
}

let styles = {
  imageContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  
  image: {
    width: 200,
    height: 200,
  },
  
  scaledUp: {
    transform: [
      { scale: 1.2 },
    ],
  },
};

...
```

This does exactly what we wanted but is violating every single principle. let's address the first priciple. This component is doing far too much. It is handling getting the data, positioning the image, and implementing our press to scale behavior. none of these rely on each other to be useful. It likely that we will want to center a piece of content elsewhere and likely that other components will require cat data data. This component is prime for destructoring. Let's break each of these into their own component.

--
###single responsibility

```js
// FlexCenter.js
...

class FlexCenter extends Component {
  render() {
    return (
      <View style={[style, this.props.style]} {...this.props} >
        {this.props.children}
      </View>
    )
  }
};

let style = {
  flex: 1,
  justifyContent: 'center',
  alignItems: 'center',
};

...
```
Here we've broken out the positioning into its own component. It passes through all its props to the View and adds only the style associated with centering. This is a layout component. It only deals with positioning the content within itself. 

--

```js
// ScaleableCatImage.js
...

class ScaleableCatImage extends Component {
  constructor(props) {
    super(props);
    
    this.state = {
      isScaledUp: false,
    }; 
  }
  
  toggleScale() {
    this.setState({
      isScaledUp: !this.state.isScaledUp,
    });
  } 
  
  render() {
    return (
      <Image
        style={[
          styles.image, 
          this.state.isScaledUp && styles.scaledUp 
        ]}
        source={{uri: this.props.cat.imageUrl}} 
        onPress={this.toggleScale.bind(this)}
      />
    )
  }

  let styles = { 
    image: {
      width: 200,
      height: 200,
    },
  
    scaledUp: {
      transform: [
        { scale: 1.2 },
      ],
    },
  }  
}

...
```
Here we have the scaleable cat image component which implements our scaling behavior. This is a UI component because it implements a descrete piece of visual interface.

--

```js
// CatContainer.js
...

let cat = {
  imageUrl: 'http://avurls.com/23443',
  ...
};

class CatContainer extends Component {
  constructor(props) {
    super(props);
    
    this.state = {
      cat: null,
    };
  }
  
  componentDidMount() {
    // normally this would be getting data from
    // a flux store or data service
    this.setState({ cat });  
  }
  
  render() {
    <FlexCenter>
      { this.state.cat ?
        <ScaleableCatImage cat={this.state.cat} />
      :
        <LoadingIndicatior />
      }
    </FlexCenter>
  }
}

...
```
this is the cat container and is a container component. it handles collecting the data needed and passing it down as props to it's children.

--

now that we've destructed this component into smaller pieces you might notice that we've introduced much more code. it's taken more effort and you might be wondering if it's worth it. the truth is, the benfits are not always immediate. However, as your application grows or your requirements change, you'll be glad to have separated your concerns.  

 In this case, our layout component is single purposed, data agnostic, and flexible. It can be hoisted out of our project and live on npm where it can be used for this project and every future project. This is even more powerful when put into the context of a company working on many different projects. Now, each project is not only achieving it's internal goals but also contributing to the wider codebase of the company. All the sudden we go from having this single layout component to resuable and flexible LayoutKit.
 
--
### data agnostic 
so we've separated our components into separate concerns. It's feeling a lot better. That is until we're tasked with making this component work with dogs as well. Dog are better the client says. regardless of our stance, we coupled our scaleable image component to cats and now we have to refactor it :(. Let's do it!

```js
// ScaleableImage.js
...

class ScaleableImage extends Component {
  constructor(props) {
    super(props);
    
    this.state = {
      isScaledUp: false,
    }; 
  }
  
  toggleScale() {
    this.setState({
      isScaledUp: !this.state.isScaledUp,
    });
  } 
  
  render() {
    return (
      <Image
        style={[
          styles.image, 
          this.state.isScaledUp && styles.scaledUp 
        ]}
        source={{uri: this.props.imageUrl}} 
        onPress={this.toggleScale.bind(this)}
      />
    )
  }

  let styles = { 
    image: {
      width: 200,
      height: 200,
    },
  
    scaledUp: {
      transform: [
        { scale: 1.2 },
      ],
    },
  }  
}

...
```
```js
// DogContainer.js
...

<ScaleableImage imageUrl={this.state.dog.imageUrl} />

...
```

Well, that wasn't too bad. It turns out that scaleable image doesn't need the cat object to properly render. It also doesn't care what type of image that it's recieving. So instead of passing the entire cat object as a property we're now sending an imageUrl. This means that it can now be used for any kind of image.  

The important piece here is figuring out what data your component actually needs to render and decoupling from the application domain. The concept of a Cat or a Dog is unimportant.

--
###flexible

Our scaleable image component is now data agnostic and we're able to use it across our app. everything is golden! ah wait. Seems like our client really likes dogs. She likes them so much that she wants them to scale up to 150%. oh, and they need to be 201px by 201px to start. and have a 5px red border because "dogs are better". Ugh. our component can't handle that. well not yet. 

```js
// ScaleableImage.js
...

class ScaleableImage extends Component {
  static defaultProps = {
    scaleUpTo: 1.2
  }

  constructor(props) {
    super(props);
    
    this.state = {
      isScaledUp: false,
    }; 
  }
  
  toggleScale() {
    this.setState({
      isScaledUp: !this.state.isScaledUp,
    });
  } 
  
  render() {
    return (
      <Image
        style={[
          styles,
          this.props.style,
          this.state.isScaledUp && { 
            transform: [{ scale: this.props.scaleUpTo }],
          }, 
        ]}
        source={{uri: this.props.imageUrl}} 
        onPress={this.toggleScale.bind(this)}
      />
    )
  }
}

let styles = {
  width: 200,
  height: 200,
};

...
```
```js
// DogContainer.js
...

<ScaleableImage 
  imageUrl={this.state.dog.imageUrl}
  scaleUpTo={1.5}
  style={styles.image} 
/>

...

let styles = {
  image: {
    width: 201,
    height: 201,
    border: '5px solid red',
  }
}

...
```

okay so we've made a number of changes to our component. First we've made the scaleUpTo value a prop and set a sensible default for when it is omitted. second, we've passed the style prop down to the Image component. Since it comes second in the style array, the passed in width and height will overwrite the defaults we've set to our style variable.

The takeaways here are:  
  1. UI components should strive to mandate as little style as possible   
  2. UI components should set reasonable defaults for their essential props / styles  
  
now comes the question of where to put our application specifc styles. The truth is, not every component can or should be reusable. Some will not adhere exactly to our three principles and will never leave the application. That's okay. For example, the container components inheriently are tied to a piece of our domain. We are unlikely to use these elsewhere and should place them in a separate directory than our reusable components.

there are two options for application specific styles:  
  
A.  pass in styles at the container level (if it is unlikely to be repeated elsewhere)  

B. wrap the component with an application namespaced component and apply the styles there.

```js
...

class MyScaleableImage extends Component {
  render() {
    return (  
      <ScaleableImage 
        style={[styles, this.props.style]}
        {...this.props} 
      />
    )
  }
}

let styles = {
  width: 201,
  height: 201,
  border: '5px solid red',
};

...
```

note: if you find yourself defining the same styles or props over and over. wrap it and use the wrapped component. It's called composition and is a core concept to React. 

--

### a last note
the core princples should invoke a mindset rather than a mandate. Time pressure will always prevent a perfect implementation. Even if you only create a handful of truely reusable components, you will be freeing yourself and others from costly and tedious reimplemention.   


--






  
  
   
 

   



 