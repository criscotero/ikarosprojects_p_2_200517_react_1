https://stackoverflow.com/questions/30721836/how-to-append-to-dom-in-react
How to append to dom in React?

Here's a js fiddle showing the question in action.

In the render function of a component, I render a div with a class .blah. In the componentDidMount function of the same component, I was expecting to be able to select the class .blah and append to it like this (since the component had mounted)

\$('.blah').append("<h2>Appended to Blah</h2>");
However, the appended content does not show up. I also tried (shown also in the fiddle) to append in the same way but from a parent component into a subcomponent, with the same result, and also from the subcomponent into the space of the parent component with the same result. My logic for attempting the latter was that one could be more sure that the dom element had been rendered.

At the same time, I was able (in the componentDidMount function) to getDOMNode and append to that

var domnode = this.getDOMNode();
\$(domnode).append("<h2>Yeah!</h2>")
yet reasons to do with CSS styling I wished to be able to append to a div with a class that I know. Also, since according to the docs getDOMNode is deprecated, and it's not possible to use the replacement to getDOMNode to do the same thing

var reactfindDomNode = React.findDOMNode();
\$(reactfindDomNode).append("<h2>doesn't work :(</h2>");
I don't think getDOMNode or findDOMNode is the correct way to do what I'm trying to do.

Question: Is it possible to append to a specific id or class in React? What approach should I use to accomplish what I'm trying to do (getDOMNode even though it's deprecated?)

var Hello = React.createClass({
componentDidMount: function(){

       $('.blah').append("<h2>Appended to Blah</h2>");
       $('.pokey').append("<h2>Can I append into sub component?</h2>");
       var domnode = this.getDOMNode();
       $(domnode).append("<h2>appended to domnode but it's actually deprecated so what do I use instead?</h2>")

       var reactfindDomNode = React.findDOMNode();
       $(reactfindDomNode).append("<h2>can't append to reactfindDomNode</h2>");

    },
    render: function() {
        return (
            <div class='blah'>Hi, why is the h2 not being appended here?
            <SubComponent/>
            </div>
        )
    }

});

var SubComponent = React.createClass({

      componentDidMount: function(){
         $('.blah').append("<h2>append to div in parent?</h2>");
      },

      render: function(){
         return(
              <div class='pokey'> Hi from Pokey, the h2 from Parent component is not appended here either?
              </div>
         )
      }

})

React.render(<Hello name="World" />, document.getElementById('container'));
javascript
reactjs
Share
Improve this question
Follow
edited Jan 30 '17 at 15:54

MPV
1,4931313 silver badges1515 bronze badges
asked Jun 9 '15 at 1:54

BrainLikeADullPencil
10.3k2222 gold badges7171 silver badges125125 bronze badges
Add a comment
3 Answers

8

In JSX, you have to use className, not class. The console should show a warning about this.

Fixed example: https://jsfiddle.net/69z2wepo/9974/

You are using React.findDOMNode incorrectly. You have to pass a React component to it, e.g.

var node = React.findDOMNode(this);
would return the DOM node of the component itself.

However, as already mentioned, you really should avoid mutating the DOM outside React. The whole point is to describe the UI once based on the state and the props of the component. Then change the state or props to rerender the component.

Share
Improve this answer
Follow
answered Jun 9 '15 at 2:18

Felix Kling
703k160160 gold badges10011001 silver badges10701070 bronze badges
if you should avoid mutating the dom, then how do people use libraries like d3.js that append to the dom? is it not a good fit? – BrainLikeADullPencil Jun 9 '15 at 2:21
Ah d3... yeah, there are exceptions to it, especially when it comes to third party libraries. If you search for React + d3 you will find a couple of approaches how this is handled. – Felix Kling Jun 9 '15 at 2:22
1
exceptions? you mean there are exceptions to the general rule that one should avoid mutating the dom? – BrainLikeADullPencil Jun 9 '15 at 2:23
3
Yep. Obviously if you want to use a component that is not React, you have to find a way to make it work. But you yourself should not perform any DOM mutations manually. Here is an example where I integrated CodeMirror (which is rather simply, to be fair): github.com/fkling/exerslide/blob/master/components/Editor.js – Felix Kling Jun 9 '15 at 2:24
@BrainLikeADullPencil @ Felix You should not mutate DOM managed by React. It's perfectly acceptable to mutate DOM outside of the element(s) you're rendering React into. If you had to use a library like d3 inside a component, it would be ok to append to the elements inside the components, as long as every time the state changes, you also apply the append the exact same way. Otherwise your appends will be overwritten and you will see inconsistencies when React re-renders the component. Also, keep in mind that if you manipulate managed DOM, React may often re-render for nothing – Codebling Dec 11 '16 at 0:05
Add a comment

6

Avoid using jQuery inside react, as it becomes a bit of an antipattern. I do use it a bit myself, but only for lookups/reads that are too complicated or near impossible with just react components.

Anyways, to solve your problem, can just leverage a state object:

<!DOCTYPE html>
<html>
  <head>
    <title></title>
    <script src="https://fb.me/react-0.13.3.js"></script>

  </head>
  <body>
    <div id='container'></div>
    <script>
    'use strict';

var Hello = React.createClass({
displayName: 'Hello',

    componentDidMount: function componentDidMount() {
        this.setState({
            blah: ['Append to blah'],
            pokey: ['pokey from parent']
        });
    },

    getInitialState: function () {
      return {
        blah: [],
        pokey: []
      };
    },

    appendBlah: function appendBlah(blah) {
        var blahs = this.state.blah;
        blahs.push(blah);
        this.setState({ blah: blahs });
    },
    render: function render() {
        var blahs = this.state.blah.map(function (b) {
            return '<h2>' + b + '</h2>';
        }).join('');
        return React.createElement(
            'div',
            { 'class': 'blah' },
            { blahs: blahs },
            React.createElement(SubComponent, { pokeys: this.state.pokey, parent: this })
        );
    }

});

var SubComponent = React.createClass({
displayName: 'SubComponent',

    componentDidMount: function componentDidMount() {
        this.props.parent.appendBlah('append to div in parent?');
    },

    render: function render() {
        var pokeys = this.props.pokeys.map(function (p) {
            return '<h2>' + p + '</h2>';
        }).join('');
        return React.createElement(
            'div',
            { 'class': 'pokey' },
            { pokeys: pokeys }
        );
    }

});

React.render(React.createElement(Hello, { name: 'World' }), document.getElementById('container'));
</script>

  </body>
</html>
Sorry for JSX conversion, but was just easier for me to test without setting up grunt :).

Anyways, what i'm doing is leveraging the state property. When you call setState, render() is invoked again. I then leverage props to pass data down to the sub component.

Share
Improve this answer
Follow
answered Jun 9 '15 at 2:08

agmcleod
12.5k1111 gold badges5252 silver badges9292 bronze badges
it's not actually jquery I'm using but another library that has an append method. (I assume your solution would still apply). I just used jquery for this question as it illustrated the problem and is easy to setup on jsfiddle.. I'll try out your answer and accept later – BrainLikeADullPencil Jun 9 '15 at 2:19
Add a comment

2

Here's a version of your JSFiddle with the fewest changes I could make: JSFiddle

agmcleod's advice is right -- avoid JQuery. I would add, avoid JQuery thinking, which took me a while to figure out. In React, the render method should render what you want to see based on the state of the component. Don't manipulate the DOM after the fact, manipulate the state. When you change the state, the component will be re-rendered and you'll see the change.

Set the initial state (we haven't appended anything).

getInitialState: function () {
return {
appended: false
};
},
Change the state (we want to append)

componentDidMount: function () {
this.setState({
appended: true
});
// ...
}
Now the render function can show the extra text or not based on the state:

render: function () {
if (this.state.appended) {
appendedH2 = <h2>Appended to Blah</h2>;
} else {
appendedH2 = "";
}
return (
<div class='blah'>Hi, why isn't the h2 being appended here? {appendedH2}
<SubComponent appended={true}/> </div>
)
}
Share
Improve this answer
Follow
edited Jun 9 '15 at 2:21
answered Jun 9 '15 at 2:18

Sean Redmond
3,5841919 silver badges2727 bronze badges
if you should avoid mutating the dom, then how do people use libraries like d3.js that append to the dom? is it not a good fit? – BrainLikeADullPencil Jun 9 '15 at 2:21
I don't know. I've used d3.js but not with React. You can manipulate the DOM, you should just not do it as part of your React components – Sean Redmond Jun 9 '15 at 2:24
