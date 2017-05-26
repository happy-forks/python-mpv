.. vim: tw=120 sw=4 et

python-mpv
==========

python-mpv is a ctypes-based python interface to the mpv media player. It gives you more or less full control of all
features of the player, just as the lua interface does.

Installation
------------

.. code:: bash

    pip install python-mpv


...though you can also realistically just copy `mpv.py`_ into your project as it's all nicely contained in one file.

Usage
-----

.. code:: python

    import mpv
    player = mpv.MPV(ytdl=True)
    player.play('https://youtu.be/DOmdB7D-pUU')


Threading
~~~~~~~~~

The ``mpv`` module starts one thread for event handling, since MPV sends events that must be processed quickly. The
event queue has a fixed maxmimum size and some operations can cause a large number of events to be sent.

If you want to handle threading yourself, you can pass ``start_event_thread=False`` to the ``MPV`` constructor and
manually call the ``MPV`` object's ``_loop`` function. If you have some strong need to not use threads and use some
external event loop (such as asyncio) instead you can do that, too with some work. The API of the backend C ``libmpv``
has a function for producing a sort of event file descriptor for a handle. You can use that to produce a file descriptor
that can be passed to an event loop to tell it to wake up the python-mpv event handler on every incoming event.

All API functions are thread-safe. If one is not, please file an issue on github.

Advanced Usage
~~~~~~~~~~~~~~

.. code:: python

    #!/usr/bin/env python3
    import mpv

    def my_log(loglevel, component, message):
        print('[{}] {}: {}'.format(loglevel, component, message))

    player = mpv.MPV(log_handler=my_log, ytdl=True, input_default_bindings=True, input_vo_keyboard=True)

    # Property access, these can be changed at runtime
    @player.property_observer('time-pos')
    def time_observer(_name, value):
        # Here, _value is either None if nothing is playing or a float containing
        # fractional seconds since the beginning of the file.
        print('Now playing at {:.2f}s'.format(value)))

    player.fullscreen = True
    player.loop = 'inf'
    # Option access, in general these require the core to reinitialize
    player['vo'] = 'opengl'

    def my_q_binding(state, key):
        if state[0] == 'd':
            print('THERE IS NO ESCAPE')
    player.register_key_binding('q', my_q_binding)

    player.play('https://youtu.be/DLzxrzFCyOs')
    player.wait_for_playback()

    del player

.. code:: python

    #!/usr/bin/env python3
    import mpv

    player = mpv.MPV(ytdl=True, input_default_bindings=True, input_vo_keyboard=True)

    player.playlist_append('https://youtu.be/PHIGke6Yzh8')
    player.playlist_append('https://youtu.be/Ji9qSuQapFY')
    player.playlist_append('https://youtu.be/6f78_Tf4Tdk')

    player.playlist_pos = 0

    while True:
        # To modify the playlist, use player.playlist_{append,clear,move,remove}. player.playlist is read-only
        print(player.playlist)
        player.wait_for_playback()

Coding conventions
------------------

The general aim is `PEP 8`_, with liberal application of the "consistency" section. 120 cells line width. Four spaces.
No tabs. Probably don't bother making pure-formatting PRs except if you think it *really* helps readability or it
*really* irks you if you don't.

.. _`PEP 8`: https://www.python.org/dev/peps/pep-0008/
.. _`mpv.py`: https://raw.githubusercontent.com/jaseg/python-mpv/master/mpv.py