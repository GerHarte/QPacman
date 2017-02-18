QPacman
=============

A simple implementation of the <a href="https://en.wikipedia.org/wiki/Q-learning">Q-Learning</a> algorithm in javascript that tries to learn how to optimally play PACMAN in an unsupervsed manner using Reinforcement Learning. 

The entire Javascript PACMAN game was developed by <a href="http://platzh1rsch.ch/"> Platzh1rsch </a> - and a more up-to-date verion is located here https://github.com/platzhersh/pacman-canvas and can be played here http://pacman.platzh1rsch.ch.


------

License
=======

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">


------

Q-Learning
===============

This repo is documenting my attempt to learn about Reinforcement Learning by trying to implement it on PACMAN. It may or may not be entirely accurate, so feel free to provide any feedback or suggestions.

Q-Learning is a Reinforcement Learning technique that allows an agent to learn the optimal action to take, given the current 'state of the world' and the set of available actions to take.

Using PACMAN as an example, a list of actions can be defined as the moves pacman can make, possible actions can be mapped in a vector A

A = [up, down, left, right, do nothing]

From the start, the game continues in cycles - at each point PACMAN needs to choose one of these actions.

As the game progresses, the 'state of the world' or just state, changes. The initial state starts off with 4 ghosts in the centre, PACMAN in the middle left and pills evenly distributed around the grid.

<img style="border-width:0" src="https://github.com/GerHarte/QPacman/blob/master/img/InitialState.png" />

As time goes on, PACMAN moves, the ghosts move and pills get eaten, the current state changes. 

<img style="border-width:0" src="https://github.com/GerHarte/QPacman/blob/master/img/ChangedState.png" />


All possible states in the game can (sort of...) be mapped in a vector S.

S = [pacman on the left in the middle, all pills intact, all ghosts in the centre box ....,
	 pacman two pixels from the left in the middle, the first three pills eaten, blinky 5 pixels above the centre box...,
	 ...]

The goal of Q-Learning is to create a table Q with one row for each state in S, and one column for each action in A. The values at each element in this table is essentially how valuable that action is (a), if you are currently in that state (s), given by a single number Q(s,a).

The intuition behind this is roughly to say, 'Look back over all the times I was in this situation before, on average what action worked out best.'


Implementation
===============

A big consideration when working with Q-Learning is how to think about the state vector. The example given above was very fuzzy, didn't describe the whole state. In reality there are an enormous number of states that even a simple game like PACMAN can be in. 

If everything else is the same, is there any real difference if PACMAN is at pixel (0,0) versus pixel (0,1)? Looking back at what action to take, Q-learning would treat these as completely separate - though you should probably do the same thing at both since they are so close together. It might make more sense to consider anything within a 10 pixel box to be the same 'state'. The beneift of this is that you have completely reduced your state-space.

Another issue with trying to identiy all the states is that there might be factors that are indifferent to location. So if a ghost directly to your left at pixel (120,140) you should move away from it, if a ghost is directly to your left at pixels (300,400) - you should also move away from it. In this case the actual location isn't relevant at all in the state.

This is what makes learning things difficult, and what makes people particularly good at it. People learn how to learn, by only taking relevant information into account when comparing against outcomes. If you're learning to play golf, you don't need to learn again any time you take a step to the left or play on a new course, you can boil it down to relevent actions you took before, and what the outcome was.

For this implementation I wanted to try and use only information that was available in the javascript objects in the game.


*Version 0.86 - 25.05.2015*
* some security fixes to avoid cheaters from adding highscores

*Version 0.84 - 09.11.2014*
* fixed bug that caused game to crash when leaving game area to the right side while holding the right arrow

*Version 0.83 - 07.05.2014*
* not possible to stop by turning into walls anymore
* mute / unmute the game by pressing the "M" key

*Version 0.82 - 02.04.2014*
* small bugfixes
* swipe gestures detection on the whole screen not only game area

*Version 0.81 - 16.03.2014*
* Ghost Modes Scatter & Chase
* Pathfinding AI for Blinky
* Ghosts need to return to Ghost House when dead

*Version 0.8 - 13.11.2013*
* lots of small changes in the backend
* when you go in landscape mode and your screen is too small to display the whole site, you get notified to rotate your phone into portrait mode
* all onClick and onMousedown in HTML removed and replaced by EventListeners in JavaScript
* Pacman Canvas now uses ApplicationCache to cache its content, so you can play the game offline!

*Version 0.78 - 05.11.2013*
* navigation via buttons should be less delayed by using onMouseDown event instead of onClick
* refreshRate is now a game attribute and could be changed easily during the game (not yet implemented in frontend)

*Version 0.77 - 24.05.2013*
* Ghosts start to blink before to undazzle
* Pacman now dies with style

*Version 0.76 - 02.05.2013*
* You can now use the usual arrow keys to control pacman
* fixed 2 small bugs regarding KeyEvents

*Version 0.75 - 28.04.2013*
* You can pause / resume the game by pressing SPACE
* ESC is no longer used to pause / resume, but to go back to the main view
* Game Menu only showing while game is paused
* some css tweaks
* Simple Highscore implemented using Ajax, Json and Sqlite3

*Version 0.74 - 25.04.2013*
* You can pause / resume the game by pressing ESC or clicking into canvas
* Swipe Gestures using hammerjs
* replaced alerts by nice html overlay messages

*Version 0.73 - 17.04.2013*
* You can play on until you lost all your 3 lives
* Ghosts state gets reset everytime they get eaten or new level starts

*Version 0.72 - 30.01.2013*
* Ghost Base Door
* Reset Game after winning

*Version 0.71 - 30.01.2013*
* Ghosts can die too

*Version 0.7 - 29.01.2013*
* Powerpills & Beastmode

*Version 0.63 - 29.01.2013*
* Pills now get loaded over external json file (map.json)
* ghost collisions implemented -> dying
* tried to clean up the code a bit

*Version 0.62 - 23.01.2013*
* disable zoom on Mobile
* change name to Pacman Canvas (Alpha)

*Version 0.61 - 12.01.2013*
* all walls defined (incl. collisions)

*Version 0.6 - 12.01.2013*
* small fixes for mobile view
* sound control (default: muted)
* collision control for walls
* json datastructure design for all game objects (pills, magic pills, walls)

*Version 0.41 - 10.12.2012*
* Mobile Design Fix
* New Icon

*Version 0.40 - 08.12.2012*
* Control Buttons for mobile
* Small Design Updates

*Version 0.30 - 05.12.2012*
* Touch Support via jGestures
* Responsive
		
*Version 0.20 - 22.11.2012*
* Code Refactored for further development
* Sound added
* Appcache implemented
	
*Version 0.13 - 29.10.2012*
* Never miss a dot: Pacman now always stays in the grid.
			
*Version 0.12 - 19.10.2012*
* Pacman is now able to eat the dots. Eating a dot equals 10 points for now.
* LiveScore implemented.
* Game ends when all dots are eaten.

*Version 0.11 - 15.10.2012*
* Placing white Dots and storing them in a Hashtable
* Monster/Ghost Prototype
* Score Prototype
* Pacman had to get smaller (r=15px)
* Display Grid
* Refactoring HTML
		
*Version 0.10 - 23.08.2012*
* Started cleaning up the code using Objects
* Pacman now turns around when changing directions
