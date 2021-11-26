We have View, Presenter and ViewState. View is interface that contains  commands that are used by Presenter. View say to Presenter that some action happened then Presenter adds command to viewState, which contains queue of commands and link to View. After reconfiguration all commands are applied accordingly to their strategies. So there are several strategies: AddToEnd, AddToEndSingle, OneExecution, SingleState, Skip
AddToEnd adds command to the end of queue
AddToEndSingle adds command to the end of queue and deletes all identical commands
OneExecution add command to the end of queue and removes it after first execution
SingleState removes all command from queue and adds command to it
SkipStrategy does not add command to queue