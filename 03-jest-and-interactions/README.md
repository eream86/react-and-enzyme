# Testing Interactions
When we are writing tests, we often want to test that the correct things happens after a user interaction.  For example, when a user clicks a button, then something appears.  Jest and Enzyme have built in tools to help us write this type of test.

## An example scenario
Let's imagine that we're on a development team that has just been given a new requirement.

    As a user when I visit the homepage I want to see a button that says "Click Me!".
    As a user, when I visit the homepage and click the "Click Me!" button, then I want to see a flash message that says "Congratulations!  You are the 1 millionth clicker of this button!"

Business is convinced that this feature is the key to unimaginable success for the company, and this experience must be flawless for each and every user that clicks the button.  We have a reputation for Test Driven Development at the company, and are the only ones they trust to build the feature. So we dive in.

## Starting with a test
We notice that the Homepage doesn't have any tests, and in fact, doesn't have a test file at all.  The convention used for this app is to put tests in the same directory as the file they are testing, so that is where we put ours.  Here's our first failing test:

```bash
cat src/pages/Home.test.js
```
```javascript
: import React from 'react';
: import ReactDOM from 'react-dom';
: import Enzyme, { mount } from 'enzyme';
: import Adapter from 'enzyme-adapter-react-16';
: Enzyme.configure({ adapter: new Adapter() });
:
: import Home from './Home'
:
: it('has a Click Me button', ()=>{
:   const home = mount(<Home />)
:   expect(home.find('button#click_me').text()).toEqual('Click Me!')
: })
```
And indeed, it fails:

```bash
PASS  src/App.test.js
FAIL  src/pages/Home.test.js
  ● has a Click Me button

    Method “text” is meant to be run on 1 node. 0 found instead.

       9 | it('has a Click Me button', ()=>{

      10 |   const home = mount(<Home />)
    > 11 |   expect(home.find('button#click_me').text()).toEqual('Click Me!')
         |                                       ^
      12 | })
      13 |

      at ReactWrapper.single (node_modules/enzyme/build/ReactWrapper.js:1700:17)
      at Object.text (src/pages/Home.test.js:11:39)

Test Suites: 1 failed, 1 passed, 2 total
Tests:       1 failed, 1 passed, 2 total
Snapshots:   0 total
Time:        1.848s, estimated 2s
Ran all test suites related to changed files.
```

## Making our first test pass

Ok, so now we're ready to make that pass.
```bash
cat src/pages/Home.js
```
```result
: import React, { Component } from 'react'
:
: class Home extends Component {
:   render() {
:     return(
:       <div>
:         <button id="click_me">Click Me!</button>
:       </div>
:     )
:   }
: }
:
: export default Home
```


```bash
 PASS  src/App.test.js
 PASS  src/pages/Home.test.js

Test Suites: 2 passed, 2 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.727s, estimated 2s
Ran all test suites related to changed files.
```
Green is good, we're getting somewhere now!


## Adding Interaction

It's time to add an interaction to the button.  We'd better make a new failing test:

```javascript
it('displays congratulations when "Click Me" button is clicked', ()=>{
  const home = mount(<Home />)
  home.find('button#click_me').simulate('click')
  expect(home.find('#flash_message').text()).toEqual('Congratulations! You are the 1 millionth clicker of this button!')
})
```

And once we see that fail, we can make it pass.  Here is the completed feature in the Home component:

```bash
cat src/pages/Home.js
```
```result
: import React, { Component } from 'react'
:
: class Home extends Component {
:   constructor(props){
:     super(props)
:     this.state = {
:       flashMessage: ''
:     }
:   }
:
:   handleClick = () => {
:     const message = 'Congratulations! You are the 1 millionth clicker of this button!'
:     this.setState({ flashMessage: message})
:   }
:
:   render() {
:     const{ flashMessage } = this.state
:     return(
:       <div>
:         <div id="flash_message">{flashMessage}</div>
:         <button
:           id="click_me"
:           onClick={this.handleClick}
:         >Click Me!</button>
:       </div>
:     )
:   }
: }
:
: export default Home
```

## Time to Refactor
Now that we have 100% test coverage of our code, we are free to refactor.  And, in fact, we take a look at our code, and decide that the ```handleClick``` function is just too wordy.  We'd rather do this as an anonymous function.  With Test coverage, we can confidently, and safely do any changes we wish.

Here's the refactored code, and our test is still green.

```bash
cat src/pages/Home.js
```
```result
: import React, { Component } from 'react'
:
: class Home extends Component {
:   constructor(props){
:     super(props)
:     this.state = {
:       flashMessage: ''
:     }
:   }
:
:   render() {
:     const{ flashMessage } = this.state
:     const message = 'Congratulations! You are the 1 millionth clicker of this button!'
:     return(
:       <div>
:         <div id="flash_message">{flashMessage}</div>
:         <button
:           id="click_me"
:           onClick={ () => this.setState({ flashMessage: message}) }
:         >Click Me!</button>
:       </div>
:     )
:   }
: }
:
: export default Home
```

```bash
 PASS  src/App.test.js
 PASS  src/pages/Home.test.js

Test Suites: 2 passed, 2 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        1.827s, estimated 2s
Ran all test suites related to changed files.
```

We're ready to submit a Pull Request for this feature.


## Exercises

You get so much admiration for nailing this feature, that your business stakeholders pile a heap of new feature requests on your team.  Here are a few of the more interesting ones.  Your job is to develop these using TDD:

* As a visitor to the homepage, I want to see a button that says "Don't click me"
* As a visitor to the homepage, when I click the "Don't click me" button, I see a flash message that says "We knew you would."
* As a visitor to the homepage, I want to be able to fill in a form with a value, and see a flash message that displays that value as I am typing.
  * Hint:  to input text in a form, try this: ```home.find('<your finder>').simulate('change', { target: { value: 'hello You!'}})```
  * Another Hint:  You can manage the value of an input field in state like this:
    ```javascript
      <input
        id="type_here"
        name="type_here"
        value={this.state.inputValue}
        onChange={(e)=> this.setState({ inputValue: e.target.value})}
      />
    ```
