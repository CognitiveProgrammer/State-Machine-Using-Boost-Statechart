# Chapter - 9 : The State Machine Contexts

_Till now we have seen state machines and state created using_ __struct__. _However, the statemachine as well as the state themselves can have the contexts of their own which needs to be accessed during different transitions._

_Boost::Statechart_ __context__ _allows accessing of statemachine and state contexts from event handlers._

# 9.1 : Accessing the context

_Let's have a two state statemachine (firstState & secondState) with its own variable called_  __stateVariable__ _which will be initialized with a value (say 100) within the statemachine constructor._

```
struct firstState;
struct secondState;

struct statemachine : sc::state_machine<statemachine, firstState> {
	int stateVariable;
	statemachine() : stateVariable(100) {
		cout << "StateMachine State Variable Constructor Value = " << stateVariable << endl;
	}
};

```
_Let's also create custom events and event handlers to handle the transiting into second state as well as looking into the state variables_

```
struct event_MoveToSecondState : sc::event<event_MoveToSecondState> {};
struct event_CheckSMVariable : sc::event<event_CheckSMVariable> {};

```
_Finally, lets create the event handlers in states to access_ __Statemachine__ _context. In_ __firstState__ _the code will change the value of_ __stateVariable__ _and in_ __secondState__ _the code will verify that the value is indeed incremented._

```
struct firstState : sc::simple_state<firstState, statemachine> {
	typedef sc::custom_reaction<event_MoveToSecondState> reactions;
	sc::result react(const event_MoveToSecondState & event) {
		context<statemachine>().stateVariable = 200;
		return transit<secondState>();
	}
};
struct secondState : sc::simple_state<secondState, statemachine> {
	typedef sc::custom_reaction<event_CheckSMVariable> reactions;
	sc::result react(const event_CheckSMVariable & event) {
		cout << "Inside event => event_CheckSMVariable | StateMachine State Variable Value = " << context<statemachine>().stateVariable << endl;
		return discard_event();
	}
};

// The Main function

int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_MoveToSecondState());
	sm.process_event(event_CheckSMVariable());
  return 0;
}
```

- NOTE : _The context<>() can't be used inside the constructor / destructors of the state_ 

