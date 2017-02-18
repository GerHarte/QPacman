QPacman
=============

A simple implementation of the <a href="https://en.wikipedia.org/wiki/Q-learning">Q-Learning</a> algorithm in javascript that tries to learn how to optimally play PACMAN in an unsupervsed manner using Reinforcement Learning. 

The entire Javascript PACMAN game was developed by <a href="http://platzh1rsch.ch/"> Platzh1rsch </a> - and a more up-to-date verion is located here https://github.com/platzhersh/pacman-canvas and can be played here http://pacman.platzh1rsch.ch.


Q-Learning
===============

This repo is documenting my attempt to learn about Reinforcement Learning by trying to implement it on PACMAN. It may or may not be entirely accurate, so feel free to provide any feedback or suggestions.

Q-Learning is a Reinforcement Learning technique that allows an agent to learn the optimal action to take, given the current 'state of the world' and the set of available actions to take.

Using PACMAN as an example, a list of actions can be defined as the moves pacman can make, possible actions can be mapped in a vector A

`A = [do nothing, up, down, left, right]`

From the start, the game continues in cycles - at each point PACMAN needs to choose one of these actions.

As the game progresses, the 'state of the world' or just state, changes. The initial state starts off with 4 ghosts in the centre, PACMAN in the middle left and pills evenly distributed around the grid.

<img style="border-width:0" src="https://github.com/GerHarte/QPacman/blob/master/img/InitialState.png" />

As time goes on, PACMAN moves, the ghosts move and pills get eaten, the current state changes. 

<img style="border-width:0" src="https://github.com/GerHarte/QPacman/blob/master/img/ChangedState.png" />


All possible states in the game can (sort of...) be mapped in a vector S.

```
S = [pacman on the left in the middle and all pills intact and all ghosts in the centre box ....,
	 pacman two pixels from the left in the middle and the first three pills eaten and blinky 5 pixels above the centre box...,
	 ...]
```

The goal of Q-Learning is to create a table Q with one row for each state in S, and one column for each action in A. The values at each element in this table is essentially how valuable that action is (a), if you are currently in that state (s), given by a single number Q(s,a).


| State        | Do Nothing           | Up  | Down  | Left  | Right  |
| ------------- |:-------------:|:-----:|:-----:|:-----:|:-----:|
| pacman on the left in the middle and all pills intact and all ghosts in the centre box ....     | 0 | 0 | 0  | 0  | 0  |
| pacman two pixels from the left in the middle and the first three pills eaten and blinky 5 pixels above the centre box... | 0 | 0 | 0  | 0  | 0  |
| ... | 0 | 0 | 0  | 0  | 0  |

The values in this Q-Table correspond roughly to a notion of rewards PACMAN gets for good and bad outcomes.

The intuition behind this is roughly to say, 'Look back over all the times I was in this situation before, on average what action worked out best.'


Implementation
===============

### Actions

Defining the actions is the easy part - as above

`A = [do nothing, up, down, left, right]`

### State Space

A big consideration when working with Q-Learning is how to think about the state vector. The example given above was very fuzzy, didn't describe the whole state. In reality there are an enormous number of states that even a simple game like PACMAN can be in. 

If everything else is the same, is there any real difference if PACMAN is at pixel (0,0) versus pixel (0,1)? Looking back at what action to take, Q-learning would treat these as completely separate - though you should probably do the same thing at both since they are so close together. It might make more sense to consider anything within a 10 pixel box to be the same 'state'. The beneift of this is that you have completely reduced your state-space.

Another issue with trying to identiy all the states is that there might be factors that are indifferent to location. So if a ghost directly to your left at pixel (120,140) you should move away from it, if a ghost is directly to your left at pixels (300,400) - you should also move away from it. In this case the actual location isn't relevant at all in the state.

This is what makes learning things difficult, and what makes people particularly good at it. People learn how to learn, by only taking relevant information into account when comparing against outcomes. If you're learning to play golf, you don't need to learn again any time you take a step to the left or play on a new course, you can boil it down to relevent actions you took before, and what the outcome was.

For a basic implementation like this we have to tell the system what the important factors to consider are, the balance is to give it enough relevant information, but not too much irrelevant information. I wanted to try and use only information that was available in the javascript objects in the game - after some trial and error I settled on the following factors to describe any given state. To simplify things I removed 3 of the ghosts and only left the Red one (blinky).

* PACMAN's absolute distance away from Blinky rounded to the nearest integer. e.g. (9)
* PACMAN's reletave location to blinky (above/below and left/right) e.g. [right, below]
* PACMAN's immediate surroundings in all 8 adjacent boxes as a string e.g. [pill, powerpill, wall, wall, nothing, wall, pill pill]
* Whether or not Blinky is in a dazzled state - we don't have to run away if is it. [true, false]

I've converted each of these features to strings, concatenated them, and then hashed them into a number. This number should act as unique id for the state (ignoring some clashes due to the <a href = "https://en.wikipedia.org/wiki/Pigeonhole_principle">Pigeonhole Principle</a>)

With some rounding, this gives roughly 30 million possible states the game can be in. In reality this is a lot lower since some states won't occur (e.g. PACMAN will never be surrounded by 8 walls).

The code to calculate the features sits in the qLearn function:
``` javascript
//Feature 1 - How far away Blinky is - Manhattan Distance
var distance_from_blinky = '(' + JSON.stringify(Math.abs(blinky_y - pacman_y) + Math.abs(blinky_x - pacman_x)) + ')'

//Feature 2 - Describe PACMAN's surroundings
var surrounding_statestring = '(' +
	JSON.stringify(game.map.posY[pacman_y-1].posX[pacman_x-1].type)+
	JSON.stringify(game.map.posY[pacman_y-1].posX[pacman_x].type)+
	JSON.stringify(game.map.posY[pacman_y-1].posX[pacman_x+1].type)+

	JSON.stringify(game.map.posY[pacman_y].posX[pacman_x-1].type)+
	JSON.stringify(game.map.posY[pacman_y].posX[pacman_x+1].type)+

	JSON.stringify(game.map.posY[pacman_y+1].posX[pacman_x-1].type)+
	JSON.stringify(game.map.posY[pacman_y+1].posX[pacman_x].type)+
	JSON.stringify(game.map.posY[pacman_y+1].posX[pacman_x+1].type) +
	')'


//Feature 3 - What direction is blinky 
var blinky_horiz_direction
if (pacman_x < blinky_x){
	blinky_horiz_direction = 'right'
} else if (pacman_x > blinky_x){
	blinky_horiz_direction = 'left'
} else {
	blinky_horiz_direction = 'same'
}

var blinky_vert_direction
if (pacman_y < blinky_y){
	blinky_vert_direction = 'below'
} else if (pacman_y > blinky_y){
	blinky_vert_direction = 'above'
} else {
	blinky_vert_direction = 'same'
}

var blinky_direction = '(' + blinky_horiz_direction +','+ blinky_vert_direction + ')'

//Feature 4 - Is blinky dazzled?
var blinky_dazzled = '(' + JSON.stringify(blinky['dazzled']) + ')'

// Concatenate all features
var statestring = blinky_direction + distance_from_blinky + surrounding_statestring + blinky_dazzled//pacman_statestring  + game_statestring + blinky_statestring //+ game_statestring;

//Remember what the last state was before setting the new state
prev_state = hashed_state;

//Hash the state string to create an ID
hashed_state = statestring.hashCode();
```

### Rewards

For rewards, I started by defining the reward PACMAN gets as just the game's score. This seemed to have a downside, that there was no feedback for taking a route with no pills, so he could get stuck circling in a local optimum. Instead I set a reward of -1 if he took a step that doesn't increase the score. That should eventually knock him out of the local optimum (I'm not sure if the discount rate in the algorithm makes this redundant...).

Other than that the rewards were closely aligned with the scores:
* Powerpill = +50
* Pill = +10
* Wall = -100 (I added this in to heavily penalise trying to turn into a wall)
* Dying = -1000 (I added to penalise dying, doesn't normally decrement your score)


### Q-learning Algotithm

Next comes the algorithm that updates our Q-Table. When an action is carried out, we update the reward element in our Q-Table based on this formula. (from Wikipedia)

<img src = "https://wikimedia.org/api/rest_v1/media/math/render/svg/7a2a11876f4a2bef1198beb780a769cfa5c21af3">

`(value of taking action a in the previous step) = (value of taking action a in the previous step) + (expected incremental value in this step)`

If `(expected incremental value in this step)` is positive, that means [action a in the previous state s] led to good things in the future, so it increases the value of [action a in the previous state s] and makes action a more likely to be picked next time we encounter state s.

In the code, the update statement is
```javascript
states[prev_state][prev_action] = old_value + alpha*(reward + discount*estimated_outcomes[action] - old_value); //[no_action,up,down,left,right]
```

The full code to calculate each part of the update statement

``` javascript
//If we haven't seen this state before, default all actions at that state to 0
if(!(hashed_state in states)){ 
	states[hashed_state] = [0,0,0,0,0]; //[no_action,up,down,left,right]
}

//Retrieve the payoff values for the current state
var estimated_outcomes = states[hashed_state]

//Get the highest payoff known for that state
var max_action_value = Math.max.apply(Math,  estimated_outcomes); 
//Get the action (or actions) corresponding to the maximum payoff
	var max_action_ids = getAllIndexes(estimated_outcomes, max_action_value)

	//Remember what the previous action was before determining the current action.
prev_action = action;

//If there is one clear winning action - do it, otherwise take a random action
if(max_action_ids.length === 1){ 
    action = max_action_ids // estimated_outcomes.indexOf(max_action_value); //find the best action for the previous state
    // console.log('Best Action Taken - ' + ['nothing', 'up', 'down', 'left', 'right'][action]);
} else {
	//When taking random actions, make it more likely to continue straight
	max_action_ids = [0,0,0,0,0,1,2,3,4] 
	//Pick a random action
    action = max_action_ids[Math.floor(Math.random()*max_action_ids.length)]; 
    // console.log('Random Action Taken - ' + ['nothing', 'up', 'down', 'left', 'right'][action]);
}

//Carry out the best action
switch (action)
{
    case 0:
        break;
    case 1:
        pacman.directionWatcher.set(up);
        break;
    case 2:
        pacman.directionWatcher.set(down);
        break;
    case 3:
        pacman.directionWatcher.set(left);
        break;
    case 4:
        pacman.directionWatcher.set(right);
        break;
}

//What was the payoff in the last state - i.e. the Old Value
old_value = states[prev_state][prev_action]
//Update Action based on the Q-Learning update formula
states[prev_state][prev_action] = old_value + alpha*(reward + discount*estimated_outcomes[action] - old_value); //[no_action,up,down,left,right]
//Reset the reward to 0
reward = 0
```

