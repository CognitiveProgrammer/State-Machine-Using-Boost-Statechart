# Chapter - 1 : Creating Basic Statemachiane using Boost::Startchart Library

## 1.1  What are StateMachines?

_State Machines are everywhere in the programming world. Almost all interactive solutions require maintaining state of the software. Though there are many ways to create state machines, boost::statechart provides one of the simplest way of creating them._

_boost::statechart takes away all the complexity of creating and handling state machines, which allows developers to focus on the functionalities and behaviour of the state._

__Before writing state machines we need to understand 3 key concepts. They are "STATES", "EVENTS" and "EVENT-HANDLERS".__

### 1.1.1  STATES

_A state is a representation of the_ __SYSTEM__ _at any given point of time. It's closely related to the literal meaning of a state. Once you know the state, you know what system holds for you in that state._

### 1.1.2  EVENTS

_An "EVENT" is a trigger that forces an activity / action in a state. The decisions like moving to another "STATE" happens because of an "EVENT". It has to be remembered that "STATES" need "EVENTS" to do any activity / action._

### 1.1.3 EVENT-HANDLERS

_An "EVENT-HANDLER" is a functionality implemented in a state for responding to an "EVENT". Its inside this "EVENT-HANDLER", the state decides to do something which may include a decision of moving to another state._

## 1.2 A simple StateMachines example with 2 states, 2 events and 2 event handlers using boost::statechart

### 1.2.1 : The required header files and namespaces

_Before using boost::statechart, we need to download and install the boost library from www.boost.org , then we need to include following headers._
```
#include <boost/statechart/event.hpp>
#include <boost/statechart/state_machine.hpp>
#include <boost/statechart/simple_state.hpp>
#include <boost/statechart/transition.hpp>
#include <boost/statechart/custom_reaction.hpp>

```
_All functionalities of boost::statechart resides in a namespaces called boost::statechart. Since we'll be using it a quiet number of times, we'll shorten the namespace via following code._

```
namespace sc = boost::statechart;

```
### 1.2.2 : Create a Boost State Machine

_Boost usage CRTP (Curiously Recursive Template Pattern) of C++ for creating states and events. To create a StateMachines we must know the starting state. After all, a statemachine can't exist without a state._
_Lets call the starting state as_ __firstState__, _then the state machine could be created as_

```
struct statemachine : sc::state_machine<statemachine, firstState> {};
```
_The statemachine needs to be derived from_ __sc::statemachine__ _which takes 2 template arguments which are_ __statemachine itself (CRTP) and the starting state__.

_Since we haven't yet defined the starting state i.e_ __firstState__, _the code will not compile. Fortunately, C++ allows forward declaration where we can declare earlier and define later. So the code above will be modified as_
```
struct firstState;
struct statemachine : sc::state_machine<statemachine, firstState> {};
```
_That's all we need to define a state machine using boost::statechart. However, the code is still not compilable as we haven't yet defined the_ __firstState__. _We have just forward declared it so as it can be used in the state machine. In the next section we'll define states__

### 1.2.3 : Creating States and associating it with State Machines

_Neither State Machines can exist with states nor states can exist without State Machines. This is particularly true about defining states and state machines with boost::statechart._

_It's for the same reason, we need to do_ __forward declaration__ _in many places while creating state machines. In the example above, the statemachine needs to know its starting state, but we cannot completely define starting state as starting state needs to know the state machine it belongs to. So here is how the state is defined_
```
struct firstState;
// State Machine need to know starting state
struct statemachine : sc::state_machine<statemachine, firstState> {};
// Starting state needs to know which state machine it belongs to
struct firstState : sc::simple_state<firstState, statemachine> {};

```
_Just like state machines, states also use CRTP patters and takes the statemachine instance as a 2nd template parameter.__

_Now this is a totally valid state machine. We can make it compile and run by writing the following code_
```
int main() {
	statemachine sm;
	sm.initiate();
	return 0;
}

```
_The code will start the statemachine which will be in its starting state_ __firstState__. _we can check the validity of the code by adding constructors in state & statemachine to make sure that it's working._

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

_A State Machine will never have a single state. At minimum, it must have two states in qualifying to be a state machine. Let's create a_ __secondState__ _which will also be the part of the existing state machine._

```
struct firstState;
// State Machine
struct statemachine : sc::state_machine<statemachine, firstState>
{
	statemachine() { cout << "Starting => statemachine" << endl; }
};
// 1st State
struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
};
// 2nd State
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
_Even though we've successfully created a second state, the output of the code will remain same as there is no way_ __secondState__ _comes into picture. In next chapters we will see how we can move from one state to another state._

_It should be noted that any state can become initial state of a state machine. In the example below, we'll make_ __secondState__ _as starting state of the state machine. For this, we need to do_ __forward declaration__ _of_ __secondState__ _before creating the_ __statemachine__.

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

_In this chapter, we learned about creating state machines with starting states and how to start the state machine_.
