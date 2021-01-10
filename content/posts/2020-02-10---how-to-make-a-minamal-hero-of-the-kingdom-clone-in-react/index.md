---
title: How to make a minimal "Hero of the Kingdom" clone in React
date: "2020-02-10T08:00:00.000Z"
template: post
draft: false
slug: "hero-of-the-kingdom-clone-react"
category: "react"
tags:
  - "react"
  - "games"
description: "Minimal clone of the \"Hero of Kingdom\" game in React"
socialImage: "/media/hotk-screenshot.jpg"
---

Over the winter holidays I was trying a lot of games to find one which suited my life situation as a parent. I wanted something with an easy to follow story but it should be good enough to keep me interested. I wanted simple gameplay but something which could get me hooked. Most importantly I wanted something which I could quit anytime and could pick up easily another time.

So I went through my Steam Library and found this gem:

![Hero of the Kingdom Screenshot](/media/hotk-screenshot.jpg)

<a href="http://lonelytroops.com/hotk/index.htm" target="_blank">Hero of the Kingdom</a> is an RPG where you can gather resources, fight enemies, solve quests and well, become the Hero. And all this can be achieved with just one mouse button. Perfect.

After I was done with the game (and the two follow-ups ;) ) I was thinking that it should be possible for me to make such a game. After a little bit of thinking I recognized that the game could be made out of a grid which in turn reminded me of a <a href="https://reactjs.org/tutorial/tutorial.htm" target="_blank">React tutorial</a> I did a while ago. I try to keep up with new technologies/frameworks (at least the most popular ones even if I don't have a use for them at the moment). The tutorial guides you through creating a *Tic-tac-toe* game with React. 

This tutorial can be turned into a minimal version of "Hero of the Kingdom" where the player can only pick up one item which is then added to his inventory. 

Why only this functionality? I've made the experience that once I concentrate on one single (important) part of a problem I found the most success. Once this is achieved extending the functionality comes much easier. 

Lets's begin.

## Concept
- **Grid:** The grid has items (which can be removed):
```
[ ][ ][ ][ ]
[ ][M][ ][ ]
[ ][ ][ ][ ]
[ ][ ][M][ ]
```
- **Player:** The player can have items (which can be added).
- **Items:** Items have types (mushrooms, herbs, etc.)
- **Interaction:** If the player clicks on grid which contains an item (in thise case "M" for mushroom) the item will be removed from grid and added to players inventory.

## Implementation
To setup the project follow on your local development environment follow this instructions: <a href="https://reactjs.org/tutorial/tutorial.html#setup-option-2-local-development-environment" target="_blank">Setup Local Development Environment for React</a> or you can use CodePen like in the tutorial.

### Changes to the tutorial code
For the most basic version where a player can pickup mushrooms (see if you can spot the mushrooms in the screenshot) and add them to its inventory it is enough to change these two functions in the `Board` component:
```javascript
class Board extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        squares: [null, 'M', null, null, null, null, null, null, 'M'], //set mushroom positions
        inventory: {
          "Mushrooms": 0,
        },
      };
    }
  
    handleClick(i) {
      if(this.state.squares[i] === 'M') { //if a mushroom is picked add it to the players inventory
        this.state.inventory["Mushrooms"]++;
      }
      this.state.squares[i] = null; //delete mushroom from square
      this.setState({squares: this.state.squares, inventory: this.state.inventory});
    }
```
![Mushroom collecting](/media/minimal-hotk-collecting.gif)
This aren't the most complex changes but this is more about the process of having a thought and taking action with the least amount of effort.

### Add item respawn timer
To make it more game-like we can add a countdown timer and respawn the mushrooms after e.g. 5 seconds. For this we can use the <a href="https://www.npmjs.com/package/react-countdown" target="_blank">react-countdown</a> module:
```
npm install react-countdown
```

First import the module and a format helper function:
```javascript
  import Countdown, {zeroPad} from 'react-countdown';
```

Add the date to the state of the `Board` constructor:
```javascript
this.state = {
  squares: [null, 'M', null, null, null, null, null, null, 'M'],
  inventory: {
    "Mushrooms": 0,
  },
  date: Date.now() + 10000,
};
```

Then add the `Countdown` component to the `render()` function of the `Board` class:
```javascript
<div>
  Time to respawn: 
  <Countdown 
    key={this.state.date} 
    date={this.state.date} 
    renderer={this.countDownRenderer}
  />
</div>
```

And finally add the `countDownRenderer` function into the body of the `Board` component:
```javascript
countDownRenderer = ({ minutes, seconds, completed }) => {
  if (completed) { //reset state
    this.state.squares = [null, 'M', null, null, null, null, null, null, 'M'];
    this.setState({ squares: this.state.squares, date: Date.now() + 5000 });
  }

  return (
    <span>
      {zeroPad(minutes)}:{zeroPad(seconds)}
    </span>
  );
};
```

Now it looks like this:
![Mushroom collecting with respawn timer](/media/minimal-hotk-collecting-with-respawn.gif)

Now we can collect mushrooms forever ;)

Of course we could take take this further but I only wanted a minimal version to get started. With this said a grid allows you to make many different types of games: You can let the player build, you can let the player command units and so on...


You can find the finished code at my <a href="https://github.com/N7Ren/minimal-hero-of-the-kingdom-clone-react" target="_blank">github repository</a> for the project.



