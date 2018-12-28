=======
API
=======
These functions are general ``asyncio`` helper functions available at the root level of the package.

..  py:decorator:: boundio.task
	boundio.asynchronous.task( *args, **kwargs )

	*Equivalent to* :py:func:`boundio.asynchronous.tasks.task`.

	Adds an asynchronous function to the global task list, with arguments specified by *args* and *kwargs*.

..  py:function:: boundio.run_tasks
	boundio.asynchronous.run_tasks( *tasks )

	*Blocking function, equivalent to* :py:func:`boundio.asynchronous.tasks.run_tasks`.

	Runs a list of tasks. Also runs all tasks marked with the :py:func:`boundio.task` decorator.

..  py:function:: boundio.execute_async
	boundio.asynchronous.execute_async( func, *args, **kwargs )

	*Asynchronous function, equivalent to* :py:func:`boundio.asynchronous.utils.execute_async`.

	Coroutine function wrapper for an arbitrary function that may or may not be a coroutine function itself.

``boundio.stdin``
-----------------
..  py:module:: boundio.stdin

This subpackage is equivalent to :py:mod:`boundio.asynchronous.raw_io.stdin`.

..  py:function:: boundio.stdin.input_async( prompt [, until='\n', encoding='utf8'] )

	*Asynchronous function, equivalent to* :py:func:`boundio.asynchronous.raw_io.stdin.input_async`

	Asynchronously reads from standard input until the sequence in ``until`` is reached.

	:param str prompt: The string to print before reading from standard input.
	:param until: The sequence that indicates that this function should return.
	:type until: str or bytes
	:param encoding: An encoding string that should be used to encode the sequence in *until* and decode the output. If encoding is ``None``, no encoding or decoding is done.
	:type encoding: str or None

	Example:

	..  code-block:: python

		from boundio.stdin import input_async
		from boundio import task, run_tasks

		@task()
		async def get_inputs():
			name = await input_async(
				'please enter your name:\n>>> ' )
			email = await input_async(
				'please enter your email:\n>>> ' )
			print(name.strip(),email.strip())

		run_tasks()

	For more examples, see :py:mod:`boundio.examples`.


.. 	function:: boundio.stdin.stdin_stream( [line_prompt='', sep='\n', encoding='utf8' ] )

	*Asynchronous generator function, equivalent to* :py:func:`boundio.asynchronous.raw_io.stdin.input_async`

	Asynchronously yields data in blocks from standard input, where each block is a sequence of characters ending with *sep*

	:param str line_prompt: The string to print before reading from standard input.
	:param until: The sequence that indicates that this function should return.
	:type until: str or bytes
	:param encoding: An encoding string that should be used to encode the sequence in *until* and decode the output. If encoding is ``None``, no encoding or decoding is done.
	:type encoding: str or None

	Example:

	..  code-block:: python

		from boundio.stdin import stdin_stream
		from boundio import task, run_tasks

		@task()
		async def echo_terminal():
			async for line in stdin_stream():
				line=line[:-1] # remove newline at end
				if line == 'quit': break
				print(line)

		run_tasks()

	For more examples, see :py:mod:`boundio.examples`.

``boundio.asynchronous``
-------------------
This subpackage includes functions for interacting with raw IO asynchronously.
..  For more information on this subpackage, see the :py:mod:`boundio.asynchronous` documentation.

``boundio.sockets``
----------------------
.. py:currentmodule:: boundio.sockets

This subpackage includes functions for reading from sockets asynchronously.
..  For more information on this subpackage, see the :py:mod:`boundio.sockets` documentation.
The following are only the public functions in the package.

..  py:function::
	boundio.sockets.get_socket_task
	boundio.sockets.tasks.get_socket_task(url [, on_open=None, on_close=None, on_message=print, time_limit=None] )

	*Asynchronous generator function.*

	Creates a socket object in a context manager and
	uses :py:func:`boundio.sockets.process_socket` to yield data.

	:param str url: Url of websocket to connect to.

	*Remaining parameters are passed directly to* :py:func:`boundio.sockets.process_socket`.

..  py:function:: boundio.sockets.get_socket_tasks
	boundio.sockets.tasks.get_socket_tasks( url, path [, on_open=None, on_close=None, on_message=print, time_limit=None] )

	Returns a *producer, consumer* awaitable pair that can be passed to :py:func:`boundio.run_tasks`. The producer
	takes information from the socket and adds it to an :py:class:`asyncio.Queue` instance. The consumer removes
	information from the queue and writes it to an output file.

	:param str url: Url of websocket to connect to.
	:param path: Path of file to write data to.
	:type path: Path, str, or other Path-like

	*Remaining parameters are passed directly to* :py:func:`boundio.sockets.process_socket`.

..  py:function:: boundio.sockets.run_socket
	boundio.sockets.tasks.run_socket(url, path [, on_open=None, on_close=None, on_message= :py:func:`boundio.sockets.process_frame`, time_limit=None] )

	*Blocking function.*

	Runs a single *producer, consumer* awaitable pair as two concurrent tasks. Convenience function to run the result
	of a single call of :py:func:`boundio.sockets.get_socket_tasks`

	:param str url: Url of websocket to connect to.
	:param path: Path of file to write data to.
	:type path: Path, str, or other Path-like

	*Remaining parameters are passed directly to* :py:func:`boundio.sockets.process_socket`.

..  py:function:: boundio.sockets.process_socket
	boundio.sockets.process.process_socket(socket [, on_open=None, on_close=None, on_message=None, time_limit=None] )

	*Asynchronous generator function.*

	Processes an open socket object, calling *on_open* at the beginning of the function call, *on_message* for
	every frame received from the socket, and *on_close* before exiting. Yields results of all functions, unless
	it is a reference to :py:const:`boundio.item_codes.SKIP_ITEM`. If this function ever receives an instance of
	:py:class:`boundio.item_codes.CLOSE_STREAM`, or a reference to the class itself, it will immediately execute
	*on_close*.

	**NOTE:** This function does not close the socket after exiting.
	You need to do that yourself or use a function that does, like :py:func:`boundio.sockets.get_socket_task`.

	:param socket: Open socket object to pull data from.
	:param on_open: Function to run on opening the socket. Takes the socket as parameters and returns text that should be yielded.
	:type on_open: callable or None
	:param on_message: Function to run each time a frame is received from the socket. Takes the socket and the frame as parameters, and returns text that should be yielded.
	:type on_message: function, async function, or None
	:param on_close: Function to run right before exiting the function.
	:type on_close: function, async function, or None
	:param time_limit: How long to keep the socket open for. If None, the socket will stay open until *on_open* or *on_message* return an instance of :py:class:`boundio.item_codes.CLOSE_STREAM`, the stream times out, or an error is raised.
	:type time_limit: int or None

..  py:function:: boundio.sockets.process_frame
	boundio.sockets.utils.process_frame( socket,frame )

	The default function for processing frames from a websocket. Returns the frame compressed to a single line with a trailing
	newline character.

``boundio.item_codes``
----------------------
This module contains objects and classes used to signal stream status. All codes except
for :py:class:`boundio.item_codes.CLOSE_STREAM` print as an empty string.

..  py:class:: boundio.item_codes.ITEM_CODE( name )

	Class to override to send messages between coroutines. The ``__str__`` method returns
	an empty string so as to prevent signals from being outputted. Constructor takes argument
	*name* to indicate the nature of the signal.

..  py:class:: boundio.item_codes.CLOSE_STREAM( frame )

	Signals that the current stream should be closed. The constructor takes argument *frame* to indicate
	the frame that triggered the CLOSE_STREAM request.

..  py:data:: boundio.item_codes.SKIP_ITEM

	Signals that the current item shouldn't be yielded.

``boundio.examples``
----------------------
This subpackage includes scripts to demonstrate how to use functions in this toolkit. Check out the source code
or import them and print the ``source`` attribute to check them out.

For example:

.. code-block:: python

	import boundio.examples.stdin_example as example
	print(example.source)
