# Chapter - 2 : Creating Events and Event handlers

_State Machines need "EVENTS" and "EVENT-HANDLERS" for doing any meaningful activity or action. In the absence of these, state machines will become dummies and does nothing. In this chapter we'll learn about giving lives to statemachine by creating "EVENTS" and "EVENT-HANDLERS"._

## 2.1 : Creating EVENTS

_A event represents an intent for the state to do something about it. Its good to know that its the "STATES" that act on "EVENTS" and not the State Machine. Here State Machine is just a logical entity which binds all the "STATES" and handles transitioning between "STATES"._

_Programmatically, the "EVENTS" are created in similar way as we create "STATES" using CRTP (Curiously Recursive Template Pattern)._

_Let's create an event on which we move between states, Lets name these events as_ __event_MoveToSecondState__ _for moving_ __secondState__ _from_ __firstState__ _and a event called_ __event_MoveToFirstState__ _for moving_ __firstState__ _from_ __secondState__.

```
struct event_MoveToSecondState : sc::event<event_MoveToSecondState> {};
struct event_MoveToFirstState : sc::event<event_MoveToFirstState> {};

```
_Rest of the code written in chapter - 1 doesn't change. However, we get nothing just by defining events. we need to handle those "EVENTS" inside the "STATES" to do some activity / action._

## 2.2 : Creating EVENT-HANDLERS
_Event handlers are the places where all actions happens. If you're creating a state machine, its likely that almost 99% or the functionality is written in event handlers._

_boost::statechart provides two way of handling events. One is_ __Automated__, _where the State Machine takes care of functionalities like moving to another state. Another one is_ __Manual__, _where we need to manually do everything including transitioning to another state.

_While creating a functional statechart, you may end up using_ __Manual__ _event handlers for >95% of time._

### 2.2.1 : Creating Automated EVENT-HANDLERS

_boost::statechart provides a inbuilt functionality called_ __sc::transition__ _which takes 2 paprameters which are_ __EVENT__ _and_ __TARGET-STATE__.

_if the_ __sc::transition__ _mentioned event is received withing the state, it will automatically transit to_ __TARGET-STATE__.

_Though it simple to use, but the syntax is bit weird at first. you need to typedef it with the reactions as_
```
typedef sc::transition <event_MoveToSecondState, secondState> reactions;

```
_if this code is present inside a state, it will handle_ __event_MoveToSecondState__ _and transit the statemachine to_ __secondState__.

```
struct firstState;
struct secondState;
struct statemachine : sc::state_machine<statemachine, firstState> {};

struct event_MoveToSecondState : sc::event<event_MoveToSecondState> {};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
	typedef sc::transition<event_MoveToSecondState, secondState> reactions;
};
struct secondState : sc::simple_state<secondState, statemachine>
{
	secondState() { cout << "In State => secondState" << endl; }
};

```
_The event is triggered by invoking the statemachine.process_event() function. The code for triggering this event is given below_

```
int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_MoveToSecondState());
	return 0;
}

```
_When we run this code, the statemachine successfully transit to_ __secondState__ _following the event_ __event_MoveToSecondState__. _Now we can also add an event  hander in_ __secondState__ _for handling event_ __event_MoveToFirstState__

```
struct firstState;
struct secondState;
struct statemachine : sc::state_machine<statemachine, firstState> {};

struct event_MoveToSecondState : sc::event<event_MoveToSecondState> {};
struct event_MoveToFirstState : sc::event<event_MoveToFirstState> {};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
	typedef sc::transition<event_MoveToSecondState, secondState> reactions;
};
struct secondState : sc::simple_state<secondState, statemachine>
{
	secondState() { cout << "In State => secondState" << endl; }
	typedef sc::transition<event_MoveToFirstState, firstState> reactions;
};

int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_MoveToSecondState());
	sm.process_event(event_MoveToFirstState());
	return 0;
}

```

_The prints in the constructors will show the flow of code based on the events triggered and execution of event handlers._

#### 2.2.1.1 : Multiple Event handlers

_A state needs to handle or respond to multiple events in its lifetime. We can have multiple_ __sc::transition__ _inside an state, but it has to be grouped under a list called_ __boost::mpl::list__. _The syntax of this is looks like_
```
namespace mpl = boost::mpl;
typedef mpl::list< sc::transition<EVENT, STATE>,
                   sc::transition<EVENT, STATE> > reactions;
```
_Lets create a new state called_ __thirdState__ _and a new event called_ __event_MoveToThirdState__ _which will trigger movement from_ __firstState__ _to_ __thirdState__ . _Now the state_ __firstState__ _can handle 2 events and move to different states based on these events being triggered._

```
struct firstState;
struct secondState;
struct thirdState;

struct statemachine : sc::state_machine<statemachine, firstState> {};

struct event_MoveToSecondState : sc::event<event_MoveToSecondState> {};
struct event_MoveToThirdState : sc::event<event_MoveToThirdState> {};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
	typedef mpl::list< sc::transition<event_MoveToSecondState, secondState>,
	sc::transition<event_MoveToThirdState, thirdState> > reactions;
};
struct secondState : sc::simple_state<secondState, statemachine>
{
	secondState() { cout << "In State => secondState" << endl; }
};
struct thirdState : sc::simple_state<thirdState, statemachine>
{
	thirdState() { cout << "In State => thirdState" << endl; }
};


int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_MoveToThirdState());
	return 0;
}

```

### 2.2.2 : Creating Manual EVENT-HANDLERS

_Manual "EVENT-HANDLERS" are the places where real code for the statemachine will be written by developers._ __Boost::statechart__ _provides a freamework to write custom event handlers. Custom events are declared using_ __sc::customer_Reaction__ _and these custom handler functions are named as_ __react__ _and will always return_ __sc::result__. _It takes the const Event reference as a parameter. The signature of the react function is fixed and can't be changed. Lets convert the above exampe to transiting the class using custom handlers._

```
struct firstState;
struct secondState;

struct statemachine : sc::state_machine<statemachine, firstState> {};

struct event_MoveToSecondState : sc::event<event_MoveToSecondState> {};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
	typedef sc::custom_reaction<event_MoveToSecondState> reactions;
	sc::result react(const event_MoveToSecondState &event) {
		return transit<secondState>();
	}
};
struct secondState : sc::simple_state<secondState, statemachine>
{
	secondState() { cout << "In State => secondState" << endl; }
};


int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_MoveToSecondState());
	return 0;
}

```
_In the state react function, we do what we want to do when the named event is received. In the example above, we're transiting to another state. However, we can choose to discard the event and remain in_ __firstState__ _by changing the code as_
```
sc::result react(const event_MoveToSecondState &event) {
		return discard_event();
	}

```

#### 2.2.2.1 : Creating Multiple Manual EVENT-HANDLERS

_Similar to automated event handlers, we can have multiple manual event handlers using_ __mpl::list__. _We do need to write saperate_ __react__ _functions for each event type. Let's reqrite the 2 event handler, one moves to_ __secondState__ _and another moves to_ __thirdState__.

```

struct firstState;
struct secondState;
struct thirdState;

struct statemachine : sc::state_machine<statemachine, firstState> {};

struct event_MoveToSecondState : sc::event<event_MoveToSecondState> {};
struct event_MoveToThirdState : sc::event<event_MoveToThirdState> {};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
	typedef mpl::list <
		sc::custom_reaction<event_MoveToSecondState>,
		sc::custom_reaction<event_MoveToThirdState>> reactions;

	sc::result react(const event_MoveToSecondState &event) {
		return transit<secondState>();
	}
	sc::result react(const event_MoveToThirdState &event) {
		return transit<thirdState>();
	}
};
struct secondState : sc::simple_state<secondState, statemachine>
{
	secondState() { cout << "In State => secondState" << endl; }
};
struct thirdState : sc::simple_state<thirdState, statemachine>
{
	thirdState() { cout << "In State => thirdState" << endl; }
};

int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_MoveToThirdState());
	return 0;
}

```

#### 2.2.2.2 : Combining Automated and Manual Event handlers

_Its possible to combine automated transition and manual event handling. This can be done by having both_ __sc::transition__ _and_ __sc::custom_reaction__ _inside the_ __mpl::list__. _Lets change the example above where transition to_ __secondState__ _shall be done automatically and transition to_ __thirdState__ _shall be done manually_.

```
struct firstState;
struct secondState;
struct thirdState;

struct statemachine : sc::state_machine<statemachine, firstState> {};

struct event_MoveToSecondState : sc::event<event_MoveToSecondState> {};
struct event_MoveToThirdState : sc::event<event_MoveToThirdState> {};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
	typedef mpl::list <
		sc::transition<event_MoveToSecondState, secondState>,
		sc::custom_reaction<event_MoveToThirdState>> reactions;

	sc::result react(const event_MoveToThirdState &event) {
		return transit<thirdState>();
	}
};
struct secondState : sc::simple_state<secondState, statemachine>
{
	secondState() { cout << "In State => secondState" << endl; }
};
struct thirdState : sc::simple_state<thirdState, statemachine>
{
	thirdState() { cout << "In State => thirdState" << endl; }
};

int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_MoveToSecondState());
	return 0;
}
```

## 2.3 : Conclusion

_In this chapter, we learned how to make state machines behaves in a dynamic way by creating "EVENTS" and "EVENT_HANDLERS"._