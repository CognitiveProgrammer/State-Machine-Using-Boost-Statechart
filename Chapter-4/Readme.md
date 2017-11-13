# Chapter - 4 : Working with Meta States

_A functional state machine mainly consists of_ __Meta States__. _A Meta State is a State with multiple inner state. The State can itself act as a mini state machine._

_Meta States are fundamental to state machine design. In a real state machine, meta states are more common than in education / learning state machines._

_For example, if a e-commerce website_ __shopping cart__ _is a state, then every time an item it added, or removed, or updated in quantity, will lead to a different state within the Meta_ __Shopping Cart State__.


## 4.1  Defining Meta States

_From the perspective of_ __boost::statechart__ _library, there is no functional difference between a_ __Simple State__ _and a_ __Meta State__. _Both behave in a similar manner and shall be capable of handling Automated and Manual events._

_Just like state machines, which needs to know the first state of the state machine, a Meta State also needs to know the first state of that meta state._

_We do specify this while creating the Meta State_.

_Let's see an example where we'll be creating_ __3 Inner States__ of a Meta State_ __firstState__.

```
struct firstState; // The Meta State
struct secondState;

// Inner States of Meta State => "firstState"
struct firstState_Inner_1;
struct firstState_Inner_2;
struct firstState_Inner_3;

```
_As earlier, we need to specify the starting state while creating the state machine, which will be_ __firstState__ .

```
struct statemachine : sc::state_machine<statemachine, firstState> {};

```
_Now, the create Meta State, we need to also specify the starting state of the_ __firstState__. _This is done by providing a 3rd parameter while creating the state._

```
struct firstState : sc::simple_state<firstState, statemachine, firstState_Inner_1> {};

```
_In the code above, the_ __firstState_Inner_1__ _becomes the starting state of Meta State_ __firstState__.

_We can go on further to define the_ __firstState_Inner_1__ _like a normal state as_

```
struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1, firstState> {
	firstState_Inner_1() { cout << "In State => firstState_Inner_1" << endl; }
};
```
_The only difference between a normal state and the meta state is that the normal state takes_ __statemachine__ _as a 2nd template parameter while meta state takes_ __Meta State__ _as 2nd template parameter. The stetemachine code once stated will be in in_ __firstState_Inner_1__.

## 4.2  State Transition inside Meta States

_The state transition within Meta States works in exactly similar manner as its done for outer states. Let's create an example with 3 inner states and the corresponding events to move inner states from one state to another state_.

```
struct firstState; // The Meta State

// Inner States of firstState
struct firstState_Inner_1;
struct firstState_Inner_2;
struct firstState_Inner_3;

// Inner State Movement Events
struct event_Inner1_Inner2 : sc::event<event_Inner1_Inner2> {};
struct event_Inner2_Inner3 : sc::event<event_Inner2_Inner3> {};
struct event_Inner3_Inner1 : sc::event<event_Inner3_Inner1> {};

// Defining the State Machine
struct statemachine : sc::state_machine<statemachine, firstState>{};

// Defining the Meta State
struct firstState : sc::simple_state<firstState, statemachine, firstState_Inner_1> {};

// The 3 Inner States of the Meta State
struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1, firstState> {
	firstState_Inner_1() { cout << "In State => firstState_Inner_1" << endl; }
	typedef sc::transition<event_Inner1_Inner2, firstState_Inner_2> reactions;
};

struct firstState_Inner_2 : sc::simple_state<firstState_Inner_2, firstState> {
	firstState_Inner_2() { cout << "In State => firstState_Inner_2" << endl; }
	typedef sc::transition<event_Inner2_Inner3, firstState_Inner_3> reactions;
};

struct firstState_Inner_3 : sc::simple_state<firstState_Inner_3, firstState> {
	firstState_Inner_3() { cout << "In State => firstState_Inner_3" << endl; }
	typedef sc::transition<event_Inner3_Inner1, firstState_Inner_1> reactions;
};

// The main functiona

int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_Inner1_Inner2());
	sm.process_event(event_Inner2_Inner3());
	sm.process_event(event_Inner3_Inner1());

	return 0;
}

```
_In this example, the Meta state moved from Inner State => 1 -> Inner State => 2 -> Inner State => 3 -> Inner State - 1. From State machine perspective there wasn't any difference in firing the events._

## 4.2  Handling Events in Meta State

_In the above example, we handled the event in Inner States of the Meta State. This doesn't prevent us from adding event handlers for the same set of events in the Meta State also. We can very well write Meta State event handler as_
```
struct firstState : sc::simple_state<firstState, statemachine, firstState_Inner_1> {
	typedef sc::custom_reaction<event_Inner1_Inner2> reactions;
	sc::result react(const event_Inner1_Inner2 & event) {
		return treturn discard_event();;
	}
};

```

_The statemachine work in such a way that if the event_ __event_Inner1_Inner2__ _is handled inside the Inner State, it will not be propagated to the Meta State. However, if the event is not handled in the Inner State, it will be propagated to the Meta State to look for a event handling mechanism. In the example below if we remove the event handler from_ __firstState_Inner_1__ _then the event will be handled at_ __firstState__

```
struct firstState : sc::simple_state<firstState, statemachine, firstState_Inner_1> {
	typedef sc::custom_reaction<event_Inner1_Inner2> reactions;
	sc::result react(const event_Inner1_Inner2 & event) {
		cout << "Discarded" << endl;
		return discard_event();
	}
};

struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1, firstState> {
	firstState_Inner_1() { cout << "In State => firstState_Inner_1" << endl; }
	// No event Handler
};

int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_Inner1_Inner2());
	sm.process_event(event_Inner2_Inner3());
	sm.process_event(event_Inner3_Inner1());

	return 0;
}

```
_This program will result in the_ __Discarded__ _event as no event transition takes place as a result of the event handling at_ __firstState__.

_Nevertheless its still possible to transit to any other Inner state from the Meta State. In the code above, if we transit the state to_ __firstState_Inner_2__ _as a result of handling event_ __event_Inner1_Inner2__ _inside Meta State, then the overall result would be the same. Here is the code_
```
struct firstState : sc::simple_state<firstState, statemachine, firstState_Inner_1> {
	typedef sc::custom_reaction<event_Inner1_Inner2> reactions;
	sc::result react(const event_Inner1_Inner2 & event) {
		return transit<firstState_Inner_2>();
	}
};

struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1, firstState> {
	firstState_Inner_1() { cout << "In State => firstState_Inner_1" << endl; }
};

struct firstState_Inner_2 : sc::simple_state<firstState_Inner_2, firstState> {
	firstState_Inner_2() { cout << "In State => firstState_Inner_2" << endl; }
	typedef sc::transition<event_Inner2_Inner3, firstState_Inner_3> reactions;
};

struct firstState_Inner_3 : sc::simple_state<firstState_Inner_3, firstState> {
	firstState_Inner_3() { cout << "In State => firstState_Inner_3" << endl; }
	typedef sc::transition<event_Inner3_Inner1, firstState_Inner_1> reactions;
};

int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_Inner1_Inner2());
	sm.process_event(event_Inner2_Inner3());
	sm.process_event(event_Inner3_Inner1());

	return 0;
}

```

_In this code, since the event_ __event_Inner1_Inner2__ is not handled within_ __firstState_Inner_1__ , _it will be propagated to the outer Meta State for handling_.

_Similarly, we can transit to another state from inside a Meta State. For example, if we have a state called_ __secondState__, _outside of meta state_ __firstState__, _we can transit to_ __secondState__ _from any one of the inner state_.

```
// Outer States
struct firstState;
struct secondState;


// Inner States of firstState
struct firstState_Inner_1;
struct firstState_Inner_2;


// Inner State Events
struct event_Inner_To_Second : sc::event<event_Inner_To_Second> {};

struct statemachine : sc::state_machine<statemachine, firstState>{};

struct firstState : sc::simple_state<firstState, statemachine, firstState_Inner_1> {

};

struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1, firstState> {
	firstState_Inner_1() { cout << "In State => firstState_Inner_1" << endl; }
	typedef sc::transition<event_Inner_To_Second, secondState> reactions;
};

struct firstState_Inner_2 : sc::simple_state<firstState_Inner_2, firstState> {
	firstState_Inner_2() { cout << "In State => firstState_Inner_2" << endl; }
};


struct secondState : sc::simple_state<secondState, statemachine> {
	secondState() { cout << "In State => secondState" << endl; }
};


int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_Inner_To_Second());
	return 0;
}

```
## 4.3 Conclusion
_In this chapter, we learned how to create a_ __Meta State__ _which can have multiple inner states with its own event handlers. We have also seen that states can move from inter state to outer state and vice versa in response to an event._

