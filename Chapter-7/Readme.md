# Chapter - 7 : Error Handling

_To handle situation, where we need to do some cleanup activities,_ __boost::statechart__ _provides some mechanisms which are being discussed below_

## 7.1  Using exit() function

_A function with a signature_ __void exit(){...}__ _could be defined for every state. This only thing to remember is that the function gets called_ __ONLY IF__ _state gets terminated because state machine is moved to another state. This function will not get called, if the state is terminated because a state machine itself is terminated_

```
// States
struct firstState;
struct secondState;

// State Events
struct event_MoveToSecond : sc::event<event_MoveToSecond> {};

// State Machines
struct statemachine : sc::state_machine<statemachine, firstState> {};

struct firstState : sc::simple_state<firstState, statemachine>
{
	firstState() { cout << "In State => firstState" << endl; }
	typedef sc::transition<event_MoveToSecond, secondState> reactions;
	void exit() { cout << "exit() called in State => firstState" << endl; }

};
struct secondState : sc::simple_state<secondState, statemachine> {
	secondState() { cout << "In State => secondState" << endl; }
	void exit() { cout << "exit() called in State => secondState" << endl; }
};

// The main
int main() {
	statemachine sm;
	sm.initiate();
	sm.process_event(event_MoveToSecond());
	return 0;
}
```
_In the above example, the_ __exit()__ _function gets called for_ __firstState__ _and not for_ __secondState__ _because_ __secondState__ _is terminated with the state machine._

__NOTE: The exit() function gets called before destructor, so it's perfectly okay to treat exit() as a class member function and do the thing(s) which is otherwise not recommended in destructors__

## 7.2 : exit() functions for Inner States

_There is no difference between the behaviour of_ __exit__ _in_ __Inner States__. _It will be called when the state is transitioned from one state to another state._

_As with_ __Outer States__, _the_ __exit()__ _functions shall not be called when the state machine is terminated._

## 7.3 : Conclusion

_In this chapter, we learnt about using the inbuilt error handling mechanism of_ __boost::statechart__
