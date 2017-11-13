# Chapter - 8 :  The Orthogonal States

Orthogonal states can be considered as a __Meta State__ which can run multiple parallel internal states, i.e they can run independently of each other. An orthogonal state is useful when when a state want to have multiple inner states.

Just like creating multiple event handlers, the construct __mpl::list__ is used to create multiple inner states.

## 8.1 : Creating a Orthogonal States

The following code creates an orthogonal state '3' Inner states, each capable of running on its own.

```
// The main state
struct firstState;

// Forward Declarator : Multiple Inner States
struct firstState_Inner_1;
struct firstState_Inner_2;
struct firstState_Inner_3;

// Creating state machine with firstState
struct statemachine : sc::state_machine<statemachine, firstState> {};

// Making firstState as orthogonal state
struct firstState : sc::simple_state<firstState, statemachine,
	mpl::list<firstState_Inner_1, firstState_Inner_2, firstState_Inner_3 >>
  {};

// Create constructors of firstState inner states
struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1, firstState::orthogonal<0>> {
	firstState_Inner_1() { cout << "In State => firstState_Inner_1" << endl; }
};

struct firstState_Inner_2 : sc::simple_state<firstState_Inner_2, firstState::orthogonal<1>> {
	firstState_Inner_2() { cout << "In State => firstState_Inner_2" << endl; }
};

struct firstState_Inner_3 : sc::simple_state<firstState_Inner_3, firstState::orthogonal<2>> {
	firstState_Inner_3() { cout << "In State => firstState_Inner_3" << endl; }
};

// Run the statemachine to see the output

int main() {
	statemachine sm;
	// This will start with 3 Initial state
	sm.initiate();
	return 0;
}

```

## 8.2 : Orthogonal State Transitions

The code above contains something unusual which we haven't used till now in our code. It is
```
struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1,firstState::orthogonal<0>>
```

A __Meta State__ which is created as __Orthogonal State__ is creates __Orthogonal Regions__. The inner states can be part of any __Orthogonal Regions__. The state transitions can happen only between orthogonal regions of a state and not between different orthogonal states.

The code above, declares that the state __firstState_Inner_1__ is part of an __Orthogonal Region__ called __<0>__.

To transition from __firstState_Inner_1__ to any other state (for example __TestState_OrthoRegion_0__ ) can happen only if the state is declared as to be part of __Orthogonal Region__ called __<0>__

```
// Create Teamp Event
struct event1 : sc::event<event1> {};
// Create another state in orthogonal Region <0>

struct TestState_OrthoRegion_0 : sc::simple_state<TestState_OrthoRegion_0, firstState::orthogonal<0>> {
	TestState_OrthoRegion_0() { cout << "In State => TestState_OrthoRegion_0" << endl; }
};

// Trigger change in state from firstState_Inner_1
struct firstState_Inner_1 : sc::simple_state<firstState_Inner_1, firstState::orthogonal<0>> {
	firstState_Inner_1() { cout << "In State => firstState_Inner_1" << endl; }
	typedef sc::transition<event1, TestState_OrthoRegion_0> reactions;
};

// Trigger the transfromation from main
int main() {
	statemachine sm;
	sm.initiate();
  sm.process_event(event1());
	return 0;
}
```

The code will initiate a transition from __firstState_Inner_1__ to __TestState_OrthoRegion_0__ in the orthogonal region __<0>__. Similar state transitions can be written for different orthogonal regions.

## 8.3 : Conclusion

In this chapter, we've learned how to create and manage orthogonal states.
