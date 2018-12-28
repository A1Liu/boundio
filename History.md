# Changelog

#### Version 0.0.2
- Refactored `boundio.asyncio` to `boundio.asynchronous`, as "`asyncio`" is the name of a dependency of this package.
- Refactored `boundio.websockets` to `boundio.sockets` as "`websockets`" is the name of a dependency of this package.
- Fixed bugs preventing import
- Added `boundio.item_codes.END_IO` yielding to `boundio.sockets.process.process_socket`
- Created a class `TaskList` in `boundio.asynchronous.tasks`
- Changed `boundio.asynchronous.tasks.task_list` from a list to an instance of `TaskList`
- Changed the `boundio.task` decorator to only add references to the functions and their arguments, and construct the coroutine on execution of `boundio.run_tasks`

#### Version 0.0.1
Added all the stuffs.
