# Chapter - 5 : Deferred Events

_In the state machine code till now, we saw that the events have to be handled by the state which is receiving the event. In cases where the state is not listening to the event, the event(s) will simply gets lost_.

_Unfortunately, in many scenarios its extremely unlikely to have a synchronous event capture mechanism. In those situations, if we aren't able to handle the event in the current state, but may probably do it in another state, then the event should be preserved not discared._

_This is achieved by using_ __sc::deferral__ _which deferred the handling of event from the current state, but the event remain alive and can be handled in the next state._

_However, it's extremely tricky to use deferring events. The behavior changes based on the way the events are raised or handled_


## 5.1  Using sc::deferral

_To use sc::deferral, we need to import one special header file of boost::statechart libary called_ __deferral.hpp__

```
#include <boost/statechart/deferral.hpp>
```

__sc::derferral__ _can be used as a part of_ __mpl::list__ _along with_ __sc::trasnsition__ _and_ __sc::custom_reactions__.

_Basically with sc::deferral, the State Machine keeps the_ __NOT HANDLED__ _events in a separate queue_. _Events in that queue are triggered every time a state is transitioned from the current state to any other state_

__sc::deferral__ _takes the name of the event which needs to be deferred or be kept in a separate queue. However, we need to watch the queue carefully to understand the behaviour_

In the code below, we'll raise an event_ __event_OutOfBlueEvent__ _which is deferred but will be automatically handled in 2nd state if a handler is written for that in 2nd state. Let's watch the event queue carefully_

```
// States
struct firstState;
struct secondState;


// Events
struct event_OutOfBlueEvent : sc::event<event_OutOfBlueEvent> {};
struct event_MoveToSecond : sc::event<event_MoveToSecond> {};

struct statemachine : sc::state_machine<statemachine, firstState>{};

struct firstState : sc::simple_state<firstState, statemachine> {
	firstState() { cout << "In State => firstState" << endl; }
	typedef mpl::list<
		sc::deferral<event_OutOfBlueEvent>,
		sc::transition<event_MoveToSecond, secondState>
	>reactions;

};

struct secondState : sc::simple_state<secondState, statemachine> {
	secondState() { cout << "In State => secondState" << endl; }
	typedef sc::custom_reaction<event_OutOfBlueEvent> reactions;
	sc::result react(const event_OutOfBlueEvent & event) {
		cout << "event_OutOfBlueEvent Tiggered in => secondState" << endl;
		return discard_event();
	}
};

int main() {
	statemachine sm;
	sm.initiate();
	// will do nothing
	sm.process_event(event_OutOfBlueEvent());
	// will change state and trigger event_OutOfBlueEvent in second state
	sm.process_event(event_MoveToSecond());
	return 0;
}

```
_Let's visualize the event queue of the above code_
__DISCLAIMER NOTE__ : __The depiction below is not the actual implementation. Its only for illustration purpose__

```
sm.process_event(event_MoveToSecond());

_______________________________________________
|              Event Queue                    |
-----------------------------------------------
|             event_OutOfBlueEvent            |  
-----------------------------------------------

```
_Since in firstState, the event_ __event_OutOfBlueEvent__ _is deferred, it will remain in the event queue when firstState exits and the state machine moves to secondState_.

_The_ __secondState__ _picks up the event from the event queue and execute the event_

## 5.2  Deferring Multiple events

_It's possible to defer multiple events using multiple_ __sc::deferral__ _with appropriate event names. Let's create a new event called_ __event_OutOfGreenEvent__ _along with existing 2 events_

```
struct event_OutOfBlueEvent : sc::event<event_OutOfBlueEvent> {};
struct event_OutOfGreenEvent : sc::event<event_OutOfGreenEvent> {};
struct event_MoveToSecond : sc::event<event_MoveToSecond> {};
```
_Now update the_ __firstState__ _for deferring outofBlue as well as outofGreen events_

```
struct firstState : sc::simple_state<firstState, statemachine> {
	firstState() { cout << "In State => firstState" << endl; }
	typedef mpl::list<
		sc::deferral<event_OutOfBlueEvent>,
		sc::deferral<event_OutOfGreenEvent>,
		sc::transition<event_MoveToSecond, secondState>
	>reactions;

};
```
_Now lets capture both the events in_ __secondState__

```
struct secondState : sc::simple_state<secondState, statemachine> {
	secondState() { cout << "In State => secondState" << endl; }
	typedef mpl::list<
		sc::custom_reaction<event_OutOfGreenEvent>,
		sc::custom_reaction<event_OutOfBlueEvent>
		> reactions;
	sc::result react(const event_OutOfBlueEvent & event) {
		cout << "event_OutOfBlueEvent Tiggered in => secondState" << endl;
		return discard_event();
	}
	sc::result react(const event_OutOfGreenEvent & event) {
		cout << "event_OutOfGreenEvent Tiggered in => secondState" << endl;
		return discard_event();
	}
};

```
_Let's trigger those events from the main function_

```
int main() {
	statemachine sm;
	sm.initiate();
	// will do nothing
	sm.process_event(event_OutOfBlueEvent());
	sm.process_event(event_OutOfGreenEvent());
	// will change state and trigger event_OutOfBlueEvent and event_OutOfGreenEvent in second state
	sm.process_event(event_MoveToSecond());
	return 0;
}

```
_The event queue at the end of state one will have two events as they were deferred_
```
_______________________________________________
|              Event Queue                    |
-----------------------------------------------
|             event_OutOfBlueEvent            |  
-----------------------------------------------
|             event_OutOfGreenEvent           |  
-----------------------------------------------
```

_Both the events will be handled in_ __secondState__.

_It will be interesting to note that the sequence of the events is maintained, i.e they will be triggered in sequence. For example the output of the main function above has_ __event_OutOfBlueEvent__ _triggered before_ __event_OutOfGreenEvent__. _If we change the sequence, then the events will be handled appropriately where_ __event_OutOfGreenEvent__ _will be triggered before_ __event_OutOfBlueEvent__.

## 5.3 : Life of Deferred Events

_Let consider a statement_
__The deferred events remain the external queue of the state machine till its handled by one of the state event handler or overwritten by new events.__

_The statement sounds complicated, but will make more sense with upcoming examples_

### 5.3.1 : Adding a thirdState

_In the example above, if we handle only event_ __event_OutOfBlueEvent__ _in_ __secondState__ _then what will happen to the event_ __event_OutOfGreenEvent__ _which was_ __deferred__ _in firstState. Furthermore, if we move to a_ __thirdState__ _as a result of event_ __event_OutOfBlueEvent__, _then the deferred state will persist for_ __thirdState__

_Lets add a_ __thirdState__ _and a handler for event_ __event_OutOfGreenEvent__

_The event queue at the end of firstState will consist of two deferred events_
```
_______________________________________________
|              Event Queue                    |
-----------------------------------------------
|             event_OutOfBlueEvent            |  
-----------------------------------------------
|             event_OutOfGreenEvent           |  
-----------------------------------------------
```
_The_ __secondState__ _will handle event_ __event_OutOfBlueEvent__, _since we're transiting to_ __thirdState__, _the event queue will contain unhandled event` which is deferred from_ __first state__

```
_______________________________________________
|              Event Queue                    |
-----------------------------------------------
|             event_OutOfGreenEvent           |  
-----------------------------------------------
```
_At_ __thirdState__, _the event_ __event_OutOfGreenEvent__ _is handled as it was maintained in the deferred queue.

Unfortunately, things get changed if we change the sequence of posting events. if we post_ __event_OutOfGreenEvent__ _before_ __event_OutOfBlueEvent__ _then_ __event_OutOfGreenEvent__ _remains unhandled_

_Let's see how event queues  plays a role in it. First, we'll amend the sequence of posting events_

```
sm.process_event(event_OutOfGreenEvent());
sm.process_event(event_OutOfBlueEvent());
```
_Since both events are deferred, the event queue at the end of_ __firstState__ _will look like_

```
_______________________________________________
|              Event Queue                    |
-----------------------------------------------
|             event_OutOfGreenEvent           |  
-----------------------------------------------
|             event_OutOfBlueEvent            |  
-----------------------------------------------

```

_At_ __secondState__, _the event_ __event_OutOfGreenEvent__ _is popped out from the_ __Event Queue__, _Since there is no handler of this event in_ __secondState__ _and the event is not marked as deferred event, it will be lost. This will be followed by popping off event_ __event_OutOfBlueEvent__ _which will be handled in_ __secondState__

_When we get into this state, the event queue is empty, so it will not execute the event handler for_ __event_OutOfGreenEvent__.

_To make sure that the event_ __event_OutOfGreenEvent__ _is available in_ __thirdState__ _we again need to defer the event in_ __secondState__ _as coded below_

```
struct secondState : sc::simple_state<secondState, statemachine> {
	secondState() { cout << "In State => secondState" << endl; }
	typedef mpl::list <
		sc::custom_reaction<event_OutOfBlueEvent>,
		sc::deferral<event_OutOfGreenEvent>
		> reactions;

	sc::result react(const event_OutOfBlueEvent & event) {
		cout << "event_OutOfBlueEvent Tiggered in => secondState" << endl;
		return transit<thirdState>();
	}

};

```
_This code will allow the event queue to again have_ __event_OutOfGreenEvent__ _at the end of second state and can be handled in third state_

## 5.4 : Conclusion

_In this chapter, we learnt how to defer the events to the next state and how the sequence of events changes its behaviour._
