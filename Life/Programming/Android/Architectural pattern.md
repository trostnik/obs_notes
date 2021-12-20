# MVI
MVI contains of Model(State), View, Intent, Action, Reducer.
State defines current state of view, it is a single source of truth and it is observed by view.
View observes state, renders it and sends Intents to ViewModel
Intent is an abstraction over user action. 
Action: Intent is mapped to Action. The purpose of this is to have different intents associated with same actions.
Reducer: component that treats the result of performing action(either return State with data or State with Exception)
