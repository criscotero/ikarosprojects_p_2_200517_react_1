How to update parent's state in React?
https://stackoverflow.com/questions/35537229/how-to-update-parents-state-in-react

My structure looks as follows:

Component 1  

 - |- Component 2


 - - |- Component 4


 - - -  |- Component 5  

Component 3
Component 3 should display some data depending on state of Component 5. Since props are immutable, I can't simply save it's state in Component 1 and forward it, right? And yes, I've read about redux, but don't want to use it. I hope that it's possible to solve it just with react. Am I wrong?

javascript
reactjs
web-deployment
Share
Improve this question
Follow
edited Oct 17 '19 at 19:56

johnborges
1,9351515 silver badges2828 bronze badges
asked Feb 21 '16 at 14:17

wklm
4,59433 gold badges1616 silver badges2424 bronze badges
28
super-easy: pass the parent-setState-Function via property to the child-component: <MyChildComponent setState={p=>{this.setState(p)}} /> In the child-component call it via this.props.setState({myObj,...}); – Marcel Ennix May 8 '18 at 21:47
<MyChildComponent setState={(s,c)=>{this.setState(s, c)}} /> if your going to use this hack make sure you support the callback. – Barkermn01 Apr 10 '19 at 14:50
7
Passing a callback to set the parent's state is a really bad practice that could lead to maintenance problems. It breaks encapsulation and it makes components 2 4 and 5 tightly coupled to 1. If walk this path then you won't be able to reuse any of these child components elsewhere. It's better you have specific props so child components could trigger events whenever something happens, then the parent component would handle that event properly. – Pato Loco Oct 4 '19 at 13:51 
@MarcelEnnix, why the curly brackets around this.setState(p) ? I tried without them and it appears to work (I'm very new to React) – Biganon Apr 10 '20 at 15:12
1
@Biganon Hmm. You are right. Sorry for that 2 extra chars :-) Maybe its because i like Curly Brackets so much. I have a Shirt printed with this Statement^^ – Marcel Ennix Apr 12 '20 at 6:48
Add a comment
18 Answers

788

For child-parent communication you should pass a function setting the state from parent to child, like this

class Parent extends React.Component {
  constructor(props) {
    super(props)

    this.handler = this.handler.bind(this)
  }

  handler() {
    this.setState({
      someVar: 'some value'
    })
  }

  render() {
    return <Child handler = {this.handler} />
  }
}

class Child extends React.Component {
  render() {
    return <Button onClick = {this.props.handler}/ >
  }
}
Expand snippet
This way the child can update the parent's state with the call of a function passed with props.

But you will have to rethink your components' structure, because as I understand components 5 and 3 are not related.

One possible solution is to wrap them in a higher level component which will contain the state of both component 1 and 3. This component will set the lower level state through props.

Share
Improve this answer
Follow
edited Nov 4 '19 at 23:07
answered Feb 21 '16 at 15:03

Ivan
14.4k66 gold badges2020 silver badges2929 bronze badges
7
why do you need this.handler = this.handler.bind(this) and not just the handler function that sets the state? – chemook78 Feb 24 '17 at 17:49
50
@chemook78 in ES6 React classes methods do not auto-bind to class. So if we don't add this.handler = this.handler.bind(this) in constructor, this inside the handler function will reference the function closure, not the class. If don't want to bind all your functions in constructor, there are two more ways to deal with this using arrow functions. You could just write the click handler as onClick={()=> this.setState(...)}, or you could use property initialisers together with arrow functions as described here babeljs.io/blog/2015/06/07/react-on-es6-plus under "Arrow functions" – Ivan Feb 24 '17 at 20:54
2
Here's an example of this in action: plnkr.co/edit/tGWecotmktae8zjS5yEr?p=preview – Tamb Jun 1 '17 at 2:03 
5
That all makes sense, why e.preventDefault ? and does that require jquery? – Vincent Buscarello Jul 3 '17 at 22:21
1
Quick question, does this disallow the assignment of a local state within the child? – ThisGuyCantEven May 2 '18 at 17:29
Show 18 more comments

54

I found the following working solution to pass onClick function argument from child to the parent component:

Version with passing a method()

//ChildB component
class ChildB extends React.Component {

    render() {

        var handleToUpdate  =   this.props.handleToUpdate;
        return (<div><button onClick={() => handleToUpdate('someVar')}>
            Push me
          </button>
        </div>)
    }
}

//ParentA component
class ParentA extends React.Component {

    constructor(props) {
        super(props);
        var handleToUpdate  = this.handleToUpdate.bind(this);
        var arg1 = '';
    }

    handleToUpdate(someArg){
            alert('We pass argument from Child to Parent: ' + someArg);
            this.setState({arg1:someArg});
    }

    render() {
        var handleToUpdate  =   this.handleToUpdate;

        return (<div>
                    <ChildB handleToUpdate = {handleToUpdate.bind(this)} /></div>)
    }
}

if(document.querySelector("#demo")){
    ReactDOM.render(
        <ParentA />,
        document.querySelector("#demo")
    );
}
Look at JSFIDDLE

Version with passing an Arrow function

//ChildB component
class ChildB extends React.Component {

    render() {

        var handleToUpdate  =   this.props.handleToUpdate;
        return (<div>
          <button onClick={() => handleToUpdate('someVar')}>
            Push me
          </button>
        </div>)
    }
}

//ParentA component
class ParentA extends React.Component { 
    constructor(props) {
        super(props);
    }

    handleToUpdate = (someArg) => {
            alert('We pass argument from Child to Parent: ' + someArg);
    }

    render() {
        return (<div>
            <ChildB handleToUpdate = {this.handleToUpdate} /></div>)
    }
}

if(document.querySelector("#demo")){
    ReactDOM.render(
        <ParentA />,
        document.querySelector("#demo")
    );
}
Look at JSFIDDLE

Share
Improve this answer
Follow
edited Dec 10 '18 at 17:27
answered Mar 2 '17 at 8:23

Roman
10.9k88 gold badges5858 silver badges6666 bronze badges
This one is good! could you explain this piece: <ChildB handleToUpdate = {handleToUpdate.bind(this)} /> Why had to bind again ? – Dane Oct 14 '17 at 13:46 
@Dane - you have to bind the context of this to be the parent so that when this is called inside the child, this refers to the parent's state and not the child's. This is closure at its finest! – Casey Mar 14 '18 at 21:37
@Casey but aren't we doing that in the constructor ? And isn't that enough ?? – Dane Mar 15 '18 at 4:18
You're totally right! I missed that. Yes, if you did it in the constructor already then you're good to go! – Casey Mar 15 '18 at 4:20
You're a legend mate! This'll keep components nicely self contained without having to be forced to create parent components to handle the state exchanges – adamj Jul 1 '18 at 0:47
Show 1 more comment

17

I want to thank the most upvoted answer for giving me the idea of my own problem basically the variation of it with arrow function and passing param from child component:

 class Parent extends React.Component {
  constructor(props) {
    super(props)
    // without bind, replaced by arrow func below
  }

  handler = (val) => {
    this.setState({
      someVar: val
    })
  }

  render() {
    return <Child handler = {this.handler} />
  }
}

class Child extends React.Component {
  render() {
    return <Button onClick = {() => this.props.handler('the passing value')}/ >
  }
}
Hope it helps someone.

Share
Improve this answer
Follow
answered Feb 7 '19 at 5:53

Ardhi
2,1611717 silver badges2525 bronze badges
whats special about the arrow function over direct call ? – Ashish Kamble May 20 '19 at 5:50
@AshishKamble the this in arrow functions refers to the parent's context (i.e. Parent class). – CPHPython Jun 11 '19 at 13:14
This is a duplicate answer. You could add a comment to the accepted answer and mention about this experimental feature to use arrow function in class. – Arashsoft Aug 28 '19 at 15:47
Add a comment

9

I found the following working solution to pass onClick function argument from child to the parent component with param:

parent class :

class Parent extends React.Component {
constructor(props) {
    super(props)

    // Bind the this context to the handler function
    this.handler = this.handler.bind(this);

    // Set some state
    this.state = {
        messageShown: false
    };
}

// This method will be sent to the child component
handler(param1) {
console.log(param1);
    this.setState({
        messageShown: true
    });
}

// Render the child component and set the action property with the handler as value
render() {
    return <Child action={this.handler} />
}}
child class :

class Child extends React.Component {
render() {
    return (
        <div>
            {/* The button will execute the handler function set by the parent component */}
            <Button onClick={this.props.action.bind(this,param1)} />
        </div>
    )
} }
Share
Improve this answer
Follow
answered Aug 23 '17 at 18:11

H. parnian
10111 silver badge55 bronze badges
2
Can anyone tell whether that's an acceptable solution (particularly interested in passing the parameter as suggested). – ilans Nov 26 '17 at 6:52
param1 is just display on console not get assign it always assigns true – Ashish Kamble Jun 12 '19 at 8:31 
I can't speak to the quality of the solution, but this successfully passes the param for me. – James Mar 15 '20 at 4:29
Add a comment

9

I like the answer regarding passing functions around, its a very handy technique.

On the flip side you can also achieve this using pub/sub or using a variant, a dispatcher, as Flux does. The theory is super simple, have component 5 dispatch a message which component 3 is listening for. Component 3 then updates its state which triggers the re-render. This requires stateful components, which, depending on your viewpoint, may or may not be an anti-pattern. I'm against them personally and would rather that something else is listening for dispatches and changes state from the very top-down (Redux does this, but adds additional terminology).

import { Dispatcher } from 'flux'
import { Component } from 'React'

const dispatcher = new Dispatcher()

// Component 3
// Some methods, such as constructor, omitted for brevity
class StatefulParent extends Component {
  state = {
    text: 'foo'
  } 

  componentDidMount() {
    dispatcher.register( dispatch => {
      if ( dispatch.type === 'change' ) {
        this.setState({ text: 'bar' })
      }
    }
  }

  render() {
    return <h1>{ this.state.text }</h1>
  }
}

// Click handler
const onClick = event => {
  dispatcher.dispatch({
    type: 'change'
  })
}

// Component 5 in your example
const StatelessChild = props => {
  return <button onClick={ onClick }>Click me</button> 
}
The dispatcher bundles with Flux is very simple, it simply registers callbacks and invokes them when any dispatch occurs, passing through the contents on the dispatch (in the above terse example there is no payload with the dispatch, simply a message id). You could adapt this to traditional pub/sub (e.g. using the EventEmitter from events, or some other version) very easily if that makes more sense to you.

Share
Improve this answer
Follow
edited Mar 18 at 8:08
answered Feb 21 '16 at 15:18

Matt Styles
2,39211 gold badge1616 silver badges2222 bronze badges
My Reacts components are "running" in browser like in an official tutorial (facebook.github.io/react/docs/tutorial.html) I tried to include Flux with browserify, but browser says, that Dispatcher is not found :( – wklm Feb 21 '16 at 20:06
2
The syntax I used was ES2016 module syntax which requires transpilation (I use Babel, but there are others, the babelify transform can be used with browserify), it transpiles to var Dispatcher = require( 'flux' ).Dispatcher – Matt Styles Feb 22 '16 at 8:44
Add a comment

5

When ever you require to communicate between child to parent at any level down, then it's better to make use of context. In parent component define the context that can be invoked by the child such as

In parent component in your case component 3

static childContextTypes = {
        parentMethod: React.PropTypes.func.isRequired
      };

       getChildContext() {
        return {
          parentMethod: (parameter_from_child) => this.parentMethod(parameter_from_child)
        };
      }

parentMethod(parameter_from_child){
// update the state with parameter_from_child
}
Now in child component (component 5 in your case) , just tell this component that it want to use context of its parent.

 static contextTypes = {
       parentMethod: React.PropTypes.func.isRequired
     };
render(){
    return(
      <TouchableHighlight
        onPress={() =>this.context.parentMethod(new_state_value)}
         underlayColor='gray' >   

            <Text> update state in parent component </Text>              

      </TouchableHighlight>
)}
you can find Demo project at repo

Share
Improve this answer
Follow
edited May 20 '19 at 6:47

Ashish Kamble
1,52211 gold badge1616 silver badges2424 bronze badges
answered Jun 30 '16 at 6:55

Rajan Twanabashu
4,01344 gold badges4141 silver badges4747 bronze badges
I cant able to understand this answer, can you explain more about it – Ashish Kamble Jun 12 '19 at 4:34
Add a comment

5

This is how we can do it with the new useState hook. Method - Pass the state changer function as a props to the child component and do whatever you want to do with the function

import React, {useState} from 'react';

const ParentComponent = () => {
  const[state, setState]=useState('');
  
  return(
    <ChildConmponent stateChanger={setState} />
  )
}


const ChildConmponent = ({stateChanger, ...rest}) => {
  return(
    <button onClick={() => stateChanger('New data')}></button>
  )
}
Share
Improve this answer
Follow
answered Feb 13 at 14:27

moshfiqrony
49744 silver badges1212 bronze badges
Nice modern example – mylesthe.dev Feb 16 at 22:15
Add a comment

4

It seems that we can only pass data from parent to child as react promotes Unidirectional Data Flow, but to make parent update itself when something happens in its "child component", we generally use what is called a "callback function".

We pass the function defined in the parent to the child as "props" and call that function from the child triggering it in the parent component.

class Parent extends React.Component {
  handler = (Value_Passed_From_SubChild) => {
    console.log("Parent got triggered when a grandchild button was clicked");
    console.log("Parent->Child->SubChild");
    console.log(Value_Passed_From_SubChild);
  }
  render() {
    return <Child handler = {this.handler} />
  }
}
class Child extends React.Component {
  render() {
    return <SubChild handler = {this.props.handler}/ >
  }
}
class SubChild extends React.Component { 
  constructor(props){
   super(props);
   this.state = {
      somethingImp : [1,2,3,4]
   }
  }
  render() {
     return <button onClick = {this.props.handler(this.state.somethingImp)}>Clickme<button/>
  }
}
React.render(<Parent />,document.getElementById('app'));

 HTML
 ----
 <div id="app"></div>
In this example we can make data pass from SubChild -> Child -> Parent by passing function to its direct Child.

Share
Improve this answer
Follow
edited Jul 23 '19 at 2:15
user6269864
answered Apr 26 '19 at 12:29

Vishal Bisht
10922 silver badges11 bronze badge
Add a comment

3

-We can create ParentComponent and with handleInputChange method to update the ParentComponent state. Import the ChildComponent and we pass two props from parent to child component ie.handleInputChange function and count.

import React, { Component } from 'react';
import ChildComponent from './ChildComponent';

class ParentComponent extends Component {
  constructor(props) {
    super(props);
    this.handleInputChange = this.handleInputChange.bind(this);
    this.state = {
      count: '',
    };
  }

  handleInputChange(e) {
    const { value, name } = e.target;
    this.setState({ [name]: value });
  }

  render() {
    const { count } = this.state;
    return (
      <ChildComponent count={count} handleInputChange={this.handleInputChange} />
    );
  }
}
Now we create the ChildComponent file and save as ChildComponent.jsx. This component is stateless because the child component doesn't have a state. We use the prop-types library for props type checking.

import React from 'react';
import { func, number } from 'prop-types';

const ChildComponent = ({ handleInputChange, count }) => (
  <input onChange={handleInputChange} value={count} name="count" />
);

ChildComponent.propTypes = {
  count: number,
  handleInputChange: func.isRequired,
};

ChildComponent.defaultProps = {
  count: 0,
};

export default ChildComponent;
Share
Improve this answer
Follow
answered Oct 6 '18 at 10:16

Sachin Metkari
7433 bronze badges
How does this work when the child has a child that affects the prop of its parent? – Bobort Jan 28 '19 at 17:22 
Add a comment

3

I've used a top rated answer from this page many times, but while learning React, i've found a better way to do that, without binding and without inline function inside props.

Just look here:

class Parent extends React.Component {

  constructor() {
    super();
    this.state={
      someVar: value
    }
  }

  handleChange=(someValue)=>{
    this.setState({someVar: someValue})
  }

  render() {
    return <Child handler={this.handleChange} />
  }

}

export const Child = ({handler}) => {
  return <Button onClick={handler} />
}
The key is in an arrow function:

handleChange=(someValue)=>{
  this.setState({someVar: someValue})
}
You can read more here. Hope this will be useful for somebody =)

Share
Improve this answer
Follow
edited Jul 11 '19 at 18:58
answered Jul 11 '19 at 18:53

Александр Немков
3122 bronze badges
Add a comment

2

If this same scenario is not spread everywhere you can use React's context, specially if you don't want to introduce all the overhead that state management libraries introduce. Plus, it's easier to learn. But be careful, you could overuse it and start writing bad code. Basically you define a Container component (that will hold and keep that piece of state for you) making all the components interested in writing/reading that piece of data its children (not necessarily direct children)

https://reactjs.org/docs/context.html

You could also use plain React properly instead.

<Component5 onSomethingHappenedIn5={this.props.doSomethingAbout5} />
pass doSomethingAbout5 up to Component 1

    <Component1>
        <Component2 onSomethingHappenedIn5={somethingAbout5 => this.setState({somethingAbout5})}/>
        <Component5 propThatDependsOn5={this.state.somethingAbout5}/>
    <Component1/>
If this a common problem you should starting thinking moving the whole state of the application to someplace else. You have a few options, the most common are:

https://redux.js.org/

https://facebook.github.io/flux/

Basically, instead of managing the application state in your component you send commands when something happens to get the state updated. Components pull the state from this container as well so all the data is centralized. This doesn't mean can't use local state anymore, but that's a more advanced topic.

Share
Improve this answer
Follow
edited Oct 4 '19 at 14:11
answered Oct 4 '19 at 13:48

Pato Loco
1,03611 gold badge99 silver badges2626 bronze badges
Add a comment

2

Most of the answers given above are for React.Component based designs. If your are using useState in the recent upgrades of React library, then follow this answer

Share
Improve this answer
Follow
answered Jun 6 '20 at 3:57

leeCoder
75688 silver badges1414 bronze badges
Add a comment

1

so, if you want to update parent component,

 class ParentComponent extends React.Component {
        constructor(props){
            super(props);
            this.state = {
               page:0
            }
        }

        handler(val){
            console.log(val) // 1
        }

        render(){
          return (
              <ChildComponent onChange={this.handler} />
           )
       }
   }


class ChildComponent extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
             page:1
        };
    }

    someMethod = (page) => {
        this.setState({ page: page });
        this.props.onChange(page)
    }

    render() {
        return (
       <Button
            onClick={() => this.someMethod()} 
       > Click
        </Button>
      )
   }
}
Here onChange is an attribute with "handler" method bound to it's instance. we passed the method handler to the Child class component, to receive via onChange property in its props argument.

The attribute onChange will be set in a props object like this:

props ={
onChange : this.handler
}
and passed to the child component

So the Child component can access the value of name in the props object like this props.onChange

Its done through the use of render props.

Now the Child component has a button “Click” with an onclick event set to call the handler method passed to it via onChnge in its props argument object. So now this.props.onChange in Child holds the output method in the Parent class Reference and credits: Bits and Pieces

Share
Improve this answer
Follow
edited Aug 15 '19 at 15:22
answered Aug 14 '19 at 17:29

Preetham NT
1111 silver badge55 bronze badges
Sorry.. for delay, Here onChange is an attribute with "handler" method bound to it's instance. we passed the method handler to the Child class component, to receive via onChange property in its props argument. The attribute onChange will be set in a props object like this: props ={ onChange : this.handler } and passed to the child component So the Child component can access the value of name in the props object like this props.onChange Its done through the use of render props. Reference and credits: [blog.bitsrc.io/… – Preetham NT Aug 15 '19 at 15:01 
Add a comment

1

We can set parent state from child component by passing function into child component as props as below

class Parent extends React.Component{
    state = { term : ''}

     onInputChange = (event)=>{
         this.setState({term: event.target.value});
     }

     onFormSubmit= (event)=>{
         event.preventDefault();
         this.props.onFormSubmit(this.state.term);
     }

     render(){
         return (
             <Child onInputChange={this.onInputChange} onFormSubmit= 
                {this.onFormSubmit} />
         )
     }
}

   
class Child extends React.Component{

     render(){
        return (
             <div className="search-bar ui segment">
                 <form className="ui form" onSubmit={this.props.onFormSubmit}>
                    <div class="field">
                      <label>Search Video</label>
                      <input type="text" value={this.state.term} onChange= 
                           {this.props.onInputChange} />
                    </div>
                    
                 </form>
             </div>
         )
     }
}
This way child will update parent state onInputChange and onFormSubmit are props passed from parents. This can be called as and event Listeners in Child,hence state will get updated there

Share
Improve this answer
Follow
edited Feb 13 at 14:56

Connell.O'Donnell
2,7941111 gold badges2020 silver badges3939 bronze badges
answered Feb 13 at 13:32

Athul Narayanan
1122 bronze badges
Add a comment

0

This the way I do it.

type ParentProps = {}
type ParentState = { someValue: number }
class Parent extends React.Component<ParentProps, ParentState> {
    constructor(props: ParentProps) {
        super(props)
        this.state = { someValue: 0 }

        this.handleChange = this.handleChange.bind(this)
    }

    handleChange(value: number) {
        this.setState({...this.state, someValue: value})
    }

    render() {
        return <div>
            <Child changeFunction={this.handleChange} defaultValue={this.state.someValue} />
            <p>Value: {this.state.someValue}</p>
        </div>
    }
}

type ChildProps = { defaultValue: number, changeFunction: (value: number) => void}
type ChildState = { anotherValue: number }
class Child extends React.Component<ChildProps, ChildState> {
    constructor(props: ChildProps) {
        super(props)
        this.state = { anotherValue: this.props.defaultValue }

        this.handleChange = this.handleChange.bind(this)
    }

    handleChange(value: number) {
        this.setState({...this.state, anotherValue: value})
        this.props.changeFunction(value)
    }

    render() {
        return <div>
            <input onChange={event => this.handleChange(Number(event.target.value))} type='number' value={this.state.anotherValue}/>
        </div>
    }
}
Share
Improve this answer
Follow
answered Feb 7 '20 at 13:15
user11534547
Add a comment

0

As per your question, I understand that you need to display some conditional data in Component 3 which is based on state of Component 5. Approach :

State of Component 3 will hold a variable to check whether Component 5's state has that data
Arrow Function which will change Component 3's state variable.
Passing arrow function to Component 5 with props.
Component 5 has an arrow function which will change Component 3's state variable
Arrow function of Component 5 called on loading itself
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/16.6.3/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/16.6.3/umd/react-dom.production.min.js"></script>
Class Component3 extends React.Component {
    state = {
        someData = true
    }

    checkForData = (result) => {
        this.setState({someData : result})
    }

    render() {
        if(this.state.someData) {
            return(
                <Component5 hasData = {this.checkForData} />
                //Other Data
            );
        }
        else {
            return(
                //Other Data
            );
        }
    }
}

export default Component3;

class Component5 extends React.Component {
    state = {
        dataValue = "XYZ"
    }
    checkForData = () => {
        if(this.state.dataValue === "XYZ") {
            this.props.hasData(true);
        }
        else {
            this.props.hasData(false);
        }
    }
    render() {
        return(
            <div onLoad = {this.checkForData}>
                //Conditional Data
            </div>
        );
    }
}
export default Component5;
Expand snippet
Share
Improve this answer
Follow
answered Feb 13 at 14:15

Vikram Dhanwani
1133 bronze badges
Add a comment

0

Here is a short snippet to get two ways binding data.

The counter show the value from the parent and is updated from the child

class Parent extends React.Component {
  constructor(props) {
    super(props)
    this.handler = this.handler.bind(this)
     this.state = {
      count: 0
     }
  }

  handler() {
    this.setState({
      count: this.state.count + 1
    })
  }

  render() {
    return <Child handler={this.handler} count={this.state.count} />
  }
}

class Child extends React.Component {
  render() {
    return <button onClick={this.props.handler}>Count {this.props.count}</button>
  }
}

ReactDOM.render(<Parent />, document.getElementById("root"));
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/16.6.3/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/16.6.3/umd/react-dom.production.min.js"></script>

<div id="root"></div>
Expand snippet
Share
Improve this answer
Follow
answered Apr 6 at 20:08

crg
1,59611 gold badge66 silver badges2323 bronze badges
Add a comment

-4

<Footer 
  action={()=>this.setState({showChart: true})}
/>

<footer className="row">
    <button type="button" onClick={this.props.action}>Edit</button>
  {console.log(this.props)}
</footer>

Try this example to write inline setState, it avoids creating another function.