https://stackoverflow.com/questions/55147232/how-to-access-other-component-props-react

How to access other component props. (React)

Thinking that I have a component:

    <Component1>
        <Component2 value={0}/>
        {window.console.log(Component2.value)}
    </Component1>

How may I do this window.console.log works, because I have a problem trying to reach that prop.

/////
function tick() {
const element = (
<div>
<h1>Hello, world!</h1>
<h2>It is {new Date().toLocaleTimeString()}.</h2>
</div>
);
// highlight-next-line
ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
