# INST326 OOP Project 02 — Class-Based GUI To-Do List

> Course: INST326 — Object-Oriented Programming  
> Section: 0104  
> Author: Myles Sartor  
> Date: November 3, 2024  

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Features](#features)
3. [Requirements](#requirements)
4. [Installation and Setup](#installation-and-setup)
5. [How to Run](#how-to-run)
6. [Application Layout](#application-layout)
7. [How It Works](#how-it-works)
   - [Persistent Storage](#persistent-storage)
   - [Adding a Task](#adding-a-task)
   - [Marking a Task Complete](#marking-a-task-complete)
   - [Deleting a Task](#deleting-a-task)
   - [Dynamic Task List Refresh](#dynamic-task-list-refresh)
8. [Class Structure and OOP Design](#class-structure-and-oop-design)
   - [App](#app)
   - [TaskManager](#taskmanager)
   - [Menu](#menu)
9. [File I/O](#file-io)
10. [Limitations and Future Improvements](#limitations-and-future-improvements)

---

## Project Overview

This project is a desktop GUI to-do list application built with Python's Tkinter library. It follows an object-oriented design using multiple classes that separate concerns between the visual interface and the underlying task data management. Tasks are displayed as interactive checkboxes, can be added via a text entry field, and can be deleted after selection. All tasks are automatically saved to a plain text file (`tasks.txt`) and reloaded the next time the application is launched, providing persistence between sessions.

---

## Features

- Graphical user interface built with Tkinter and the `ttk` themed widget set
- Add new tasks via a text entry field and "Add Task" button
- Mark tasks as complete using interactive checkbox widgets
- Delete a selected (checked) task using the "Delete Task" button
- Automatic save to `tasks.txt` on every add or delete operation
- Automatic load from `tasks.txt` on startup — task history persists across sessions
- Entry field clears automatically after a task is added
- OOP architecture with three distinct classes: `App`, `TaskManager`, and `Menu`

---

## Requirements

- Python 3.x (3.6+ recommended)
- Standard library only — no external packages required:
  - `tkinter` (built-in with most Python distributions on Windows and macOS)
  - `os` (built-in)

> On some Linux distributions, Tkinter may need to be installed separately:
> ```bash
> sudo apt-get install python3-tk
> ```

---

## Installation and Setup

1. Download or clone the notebook file `INST326Project02_MylesSartor.ipynb`.
2. Open it in Jupyter Notebook or JupyterLab.
3. No external files are needed on first run. The `tasks.txt` file is created automatically when the first task is saved.
4. On subsequent runs, `tasks.txt` must be in the same directory as the notebook for tasks to load correctly.

---

## How to Run

Open the notebook and run the single code cell. The Tkinter window will launch as a separate desktop window. The notebook kernel must remain active while the application is in use.

To run as a standalone script, extract the code into a `.py` file:

```bash
python todo_app.py
```

The application window opens with the title "Class-based To-Do List" at a fixed size of 400x300 pixels.

---

## Application Layout

The window is divided into two vertical regions:

```
+------------------+--------------------------------------+
|                  |  [Task Entry Field               ]   |
|   Menu Frame     |  [ Add Task ]  [ Delete Task ]       |
|   (left 30%)     |                                      |
|   (reserved for  |  [ ] Task One                        |
|   future use)    |  [ ] Task Two                        |
|                  |  [ ] Task Three                      |
+------------------+--------------------------------------+
```

- The left panel (30% width) is occupied by the `Menu` frame, which is reserved for future expansion.
- The right panel (70% width) is the main interaction area containing the entry field, buttons, and the live task list.

---

## How It Works

### Persistent Storage

When the application starts, `TaskManager.load_tasks()` checks for the existence of `tasks.txt` in the working directory using `os.path.exists()`. If the file exists, it reads all non-empty lines as task strings and initializes a corresponding list of `tk.BooleanVar` objects (one per task) to track checkbox state:

```python
def load_tasks(self):
    if os.path.exists("tasks.txt"):
        with open("tasks.txt", "r") as file:
            self.tasks = [line.strip() for line in file if line.strip()]
            self.check_tasks = [tk.BooleanVar(value=False) for tasks in self.tasks]
```

Every time a task is added or deleted, `save_tasks()` is called immediately, writing the current task list back to `tasks.txt`. This ensures the file always reflects the current state of the application.

### Adding a Task

When the "Add Task" button is clicked, the `App.add_task()` method reads the text from the entry widget. If the entry is not empty, it delegates to `TaskManager.add_task()`, which appends the task string to `self.tasks` and adds a new `BooleanVar` to `self.check_tasks`, then saves. The task list display is refreshed via `update_task_list()`, and the entry field is cleared:

```python
def add_task(self):
    task = self.entry.get()
    if task:
        self.task_manager.add_task(task)
        self.update_task_list()
        self.entry.delete(0, tk.END)
```

### Marking a Task Complete

Each task is rendered as a `ttk.Checkbutton` widget linked to its corresponding `BooleanVar`. When a checkbox is toggled, the `BooleanVar` value changes to `True`. This state is used by `get_checked_task_index()` to identify which task is currently selected for deletion.

### Deleting a Task

The "Delete Task" button calls `App.delete_task()`, which calls `get_checked_task_index()` to scan all `BooleanVar` values and return the index of the first checked task. If a checked task is found, `TaskManager.delete_task(index)` removes both the task string and its associated `BooleanVar` from their respective lists and saves:

```python
def delete_task(self, index):
    if 0 <= index < len(self.tasks):
        del self.tasks[index]
        del self.check_tasks[index]
        self.save_tasks()
```

### Dynamic Task List Refresh

`update_task_list()` destroys all existing child widgets inside `tasks_frame` and recreates them from scratch on every call. This ensures the displayed list always exactly reflects the in-memory `tasks` list without the complexity of differential updates:

```python
def update_task_list(self):
    for widget in self.tasks_frame.winfo_children():
        widget.destroy()
    for i in range(len(self.task_manager.tasks)):
        task = self.task_manager.tasks[i]
        check_task = self.task_manager.check_tasks[i]
        checkbox = ttk.Checkbutton(self.tasks_frame, text=task, variable=check_task)
        checkbox.pack()
```

---

## Class Structure and OOP Design

### App

Inherits from `tk.Tk`, making the application window itself an object. Responsible for:
- Initializing the main window (title, size, minimum size)
- Creating and placing all top-level UI frames and widgets
- Handling all user interaction events (button clicks, input retrieval)
- Delegating data operations to `TaskManager`

### TaskManager

A standalone data-layer class with no GUI dependencies. Responsible for:
- Maintaining the in-memory lists of task strings (`self.tasks`) and their checkbox states (`self.check_tasks`)
- All file I/O: reading from and writing to `tasks.txt`
- Adding and deleting tasks from the internal lists

This separation means the data logic is decoupled from the display logic, following the principle of encapsulation and making each class independently testable.

### Menu

Inherits from `ttk.Frame`. Currently a placeholder that occupies the left 30% of the window. Its `create_widgets()` method returns immediately without creating any content. It is included in the design to support future menu functionality (e.g., filter views, settings, or task categories) without requiring structural changes to the application layout.

---

## File I/O

| File        | Format      | Contents                                            | When Written           |
|-------------|-------------|-----------------------------------------------------|------------------------|
| `tasks.txt` | Plain text  | One task per line, UTF-8 encoded, no headers        | On every add or delete |

The file is created automatically on the first task addition. If deleted manually between sessions, the application starts with an empty task list on next launch.

---

## Limitations and Future Improvements

- Only one task can be selected for deletion at a time. Multi-select deletion would improve usability for bulk operations.
- Completed tasks are visually indistinguishable from incomplete ones (the checkbox state is tracked but no visual strikethrough or color change is applied). A completed task style would improve clarity.
- The `Menu` frame is entirely unused. Future versions could add filter buttons (e.g., "Show completed", "Show incomplete") or a task category system.
- Tasks have no due dates, priority levels, or categories. Adding these fields would require a more structured data format than a plain text file (e.g., JSON or SQLite).
- There is no confirmation dialog before deleting a task, which makes accidental deletions irreversible within the session.
- The window size is fixed at 400x300 and does not scale well with a large number of tasks. Wrapping the task list in a scrollable canvas would improve usability for longer lists.
