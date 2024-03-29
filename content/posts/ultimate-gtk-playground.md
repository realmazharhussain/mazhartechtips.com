---
title: "Ultimate Inspector/Playground for GTK"
date: 2023-10-11T20:13:00+05:00
draft: false
typora-root-url: ..
---

Have you ever been in a situation where you wanted to read/inspect some value that GTK Inspector does not show or modify something in your running app e.g. replace a widget with another one? Well! I have just the trick for you (if your app is written in an interpreted language that is). 

## The Trick

The trick is to start/run the app in a background thread of the interpreter while the main thread is showing a prompt for you to type and run more code in.

## Steps

1. Make sure the file/module that contains `Application.run()` call can be imported in live interpreter mode (running the interpreter directly)
1. Replace the `Application.run()` call with `Gio.Task.run_in_thread()` call passing a wrapper around `Application.run` as `task_func` argument.
1. Run the interpreter.
1. Import the file/module that contains `Gio.Task.run_in_thread()` call.
1. If the `Gio.Task.run_in_thread()` call is inside another function, call that function. Otherwise, do nothing.

The app will start running but you'll still be able to input more code into the interpreter prompt and read/modify any part of your app while it is running.


## Example (Python)

1. Save the following code as `bg_app.py`.

   ```python
   import gi
   gi.require_version("Gtk", "4.0")

   from gi.repository import Gtk, Gio
   
   task = Gio.Task()
   app = Gtk.Application()
   win = Gtk.ApplicationWindow()
   
   def on_activate(app):
       app.add_window(win)
       win.present()

   def run_app(*args):
       app.run()
   
   app.connect('activate', on_activate)
   task.run_in_thread(run_app)
   ```

1. Open the Python interpreter in the same directory where you saved `bg_app.py`.

1. Enter the following line.

   ```python
   from bg_app import app, win, Gtk
   ```
   
   This will launch the app without blocking the interpreter.

1. Enter the following line and see the magic.

   ```python
   win.props.child = Gtk.Label.new("Wow, Magic!")
   ```
