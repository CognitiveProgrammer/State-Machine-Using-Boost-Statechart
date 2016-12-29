# Chapter - 1 : Creating Basic Statemachiane using Boost::Startchart Library 

## 1.1  What are StateMachines?

_State Machines are everywhere in the programming world. Almost all interactive solutions requires maintaining a state of the software. Though there are many ways to create state machines, boost::statechart provides one of the simplest way of creating them._

_boost::statechart takes away all the complexity of creating and handling state machines, which allows developers to focus on functionalities and behaviour of the state._

__Before writing state machines we need to understand 2 key concept. They are "STATES", "EVENTS" and "EVENT-HANDLERS".__

### 1.1.1  STATES

_A state is a representation of the system at any given point of time. Its closely related to the litral meaning of a state. Once you know the state, you know what system holds for you in that state._

### 1.1.2  EVENTS

_An "EVENT" is a trigger that forces an activity / action in a state. The decisions like moving to another "STATE" happens because of an "EVENT". It has to be remember that "STATES" need "EVENTS" to do any activity / action._

### 1.1.3 EVENT-HANDLERS

_An "EVENT-HANDLER" is a functionality implemented in a state to in response to an "EVENT". Its inside this "EVENT-HANDLER", the state decides to do something including a decision of moving to another state._

## 1.2 A simple StateMachines with 2 states, 2 events and 2 event handlers using boost::statechart

### 1.2.1 : The required header files and namespaces

_Before using boost::statechart, we need to downloal and install boost library from www.boost.org , then we need to include following handlers._
```
#include <boost/statechart/event.hpp>
#include <boost/statechart/state_machine.hpp>
#include <boost/statechart/simple_state.hpp>
#include <boost/statechart/custom_reaction.hpp>

```
_All functionalities of boost::statechart resides in a namespaces called boost::statechart. since we'll be using it quiet number of times, we'll shorten it via following code._

```
namespaces sc = boost::statechart;

```
### 1.2.2 : Create a Boost State Machine

_Boost usage CRTP (Curiously Recursive Template Pattern) of C++ for creating states and events. To create a StateMachines we must know the starting state. After all, a statemachine can't exist without a state._
_Lets call the state as_ __firstState__

```
struct statemachine : sc::state_machine<statemachine, firstState> {};
```
_The statemachine needs to be derived from_ __sc::statemachine__ _which takes the template 2 arguments which are_ __statemachine itself and starting state__.

_The first template argument is provided as what we call CRTP (Curiously Recurring Template Pattern)_

_Now we haven't defined the starting state i.e_ __firstState__ _yet, which means comiler error. Fortunately, C++ allows forward declaration where we can declare earlier and define later. so the code above will be modified as_
```
struct firstState;
struct statemachine : sc::state_machine<statemachine, firstState> {};
```
_That's all we need to define a state machine using boost::statechart. However, the code is still not compilable as we haven't yet defined the_ __firstState__ _we just did the forward declaration. Let's see in next section about how to define states__

### 1.2.3 : Creating States and associating it with State Machines

_Neither State Machines can exists with states nor states can exits without State Machines. This is particularly true about defining states and state machines with boost::statechart._

_It's for the same reason, we need to do_ __forward declaration__ _in many places while creating state machines. In the example above, the statemachine needs to know its starting state, but we can not completely define starting state as starting state needs to know the state machine it belongs to. So here is how the state is defined_
```
struct firstState;
struct statemachine : sc::state_machine<statemachine, firstState> {};
struct firstState : sc::simple_state<firstState, statemachine> {};

```
_just like state machines, states also use CRTP patters and takes the statemachine instance as a 2nd paramter.__

_Now this is a totally valid state machine. we can make it run by writing the following code_
```
int main() {
	statemachine sm;
	sm.initiate();
	return 0;
}

```
_The code will start the statemachine which will be in its starting state_ __firstState__. _we can check the validity of the code by adding constructors in state & statemachine to make sure that its working._

```
struct firstState;
struct statemachine : sc::state_machine<statemachine, firstState>
{
	statemachine() { cout << "Starting => statemachine" << endl; }
};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
};

int main() {
	statemachine sm;
	sm.initiate();
	return 0;
}

```
### 1.2.4 : Creating multiple States for a State Machines

_A State Machine will never have a single state. At mimimum, it must have two states for qualifying to be a state machine. Let's create a_ __secondState__ _which will also be the part of existing state machine._

```
struct firstState;
struct statemachine : sc::state_machine<statemachine, firstState>
{
	statemachine() { cout << "Starting => statemachine" << endl; }
};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
};
struct secondState : sc::simple_state<secondState, statemachine>
{
	secondState() { cout << "In State => secondState" << endl; }
};

int main() {
	statemachine sm;
	sm.initiate();
	return 0;
}
```
_Even though we've successfully created second state, the output of the code will remain same. In next chapters we will see how to move from one state to another state._

_Any state can become initial state of a state machine. In the example above, we can make_ __secondState__ _as starting state of the state machine. For this, we need to do_ __forward declaration__ _of_ __secondState__ _before creating the_ __statemachine__.

```
struct firstState;
struct secondState;
struct statemachine : sc::state_machine<statemachine, secondState>
{
	statemachine() { cout << "Starting => statemachine" << endl; }
};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
};
struct secondState : sc::simple_state<secondState, statemachine>
{
	secondState() { cout << "In State => secondState" << endl; }
};

int main() {
	statemachine sm;
	sm.initiate();
	return 0;
}

```

## 1.3 : Conclusion

_In this chapter we learned about creating state machine with starting states and running the state machines.

