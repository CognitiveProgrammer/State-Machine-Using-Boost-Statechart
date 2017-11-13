# Chapter - 6 : Forwarding Events

_As we saw in_ __[chapter - 4 : Working with Meta States](https://github.com/9lean/State-Machine-Using-Boost-Statechart/tree/master/Chapter-4)__, _if an event is  unhandled in_ __Inner State__, _it gets automatically propagated to_ __Outer State__. _Which means all unhandled states can be handled in_ __Outer State__.

_This is good unless we come to a situation where we want only certain events to be propagated to_ __Outer State__ _because we want to do something as part of this event handling in_ __Inner State__ _as well as in__ __Outer state__

_We can use_ __forward_even()__ _for these situations


## 6.1  Using forward_event

_Let's construct a state with an inner state, which will forward the event to outer state_

```
struct firstState;

// Inner States of firstState
struct firstState_Inner_1;

// Inner State Events that will be  forwarded
struct event_First_Inner_1 : sc::event<event_First_Inner_1> {};

// The State Machine
struct statemachine : sc::state_machine<statemachine, firstState> {};

// First State
struct firstState : sc::simple_state<firstState, statemachine, firstState_Inner_1>
{
	firstState() { cout << "In State => firstState" << endl; }

	typedef sc::custom_reaction<event_First_Inner_1> reactions;

	sc::result react(const event_First_Inner_1 & event) {
		cout << "Received Event @ Outer State => firstState" << endl;
		return discard_event();
	}
};

// Inner State
struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1, firstState> {
	firstState_Inner_1() { cout << "In State => firstState_Inner_1" << endl; }
	typedef sc::custom_reaction<event_First_Inner_1> reactions;
	sc::result react(const event_First_Inner_1 & event) {
		cout << "Forwarding Event => event_First_Inner_1 => From firstState_Inner_1" << endl;
		return forward_event();
	}
};

// The Main
int main() {
	statemachine sm;
	sm.initiate();
	// Inner state Events
	sm.process_event(event_First_Inner_1());
	return 0;
}

```
_As the output of the above program shows that, the event_ __event_First_Inner_1__ _first gets handled in the_ __Inner State__ _where it forwards the same, which is then handled by_ __Outer State__ .

_The same behaviour can be repetitive, if we trigger the event more than once using the code_

```
	sm.process_event(event_First_Inner_1());
	sm.process_event(event_First_Inner_1());

```

__forward_event()__ _also provides us the flexibility of choosing, which all events are allowed to be propagate to outer state and which are not_

## 6.2 : Conclusion

_In this chapter, we learnt how to forward the events from an inner state to outer state selectively. We also know from the code that the same event could be handled twice, once in_ __Inner State__ _and then inside_ __Outer State__
