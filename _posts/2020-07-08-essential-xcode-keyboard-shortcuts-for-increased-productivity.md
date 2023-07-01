---
layout: post
title: "Essential Xcode keyboard shortcuts for increased productivity"
date: 2020-07-08
categories: xcode
tags: xcode keyboard-shortcuts productivity tips tricks
image:
    path: /assets/images/xcode-keyboard-shortcuts.webp
---

As iOS developers, we spend a considerable amount of time in Xcode, so it’s worth taking some time to know your way around Xcode.

Knowing how to edit, search effectively, and how to navigate your project in Xcode goes a long way to increase your productivity.

Taking the time to memorize and practice these shortcuts will save you a lot of time having to reach for your mouse and will allow you to keep both your hands on the keyboard, more often.

Let's have a look at a few essential keyboard shortcuts which is easy to remember yet will save you a lot of time once you've memorized them.

## Editor shortcuts

We spend the majority of our time in Xcode using the editor, writing code. So it only makes sense to know a couple of basic keyboard shortcuts when writing and editing your code.

### Moving lines up and down (⌥ + ⌘ + [ and ])

For the longest time I was selecting the lines of code, cutting it, go to the line where I want to move it to and paste it there. Somewhere in between also required some backspacing and entering to make space for the lines pasted.

With these two shortcuts, you can do it all in one go, a great time saver especially rearranging and moving code around in SwiftUI.

![image](/assets/images/xcode-kb-shortcut-move-lines.gif)

### Balance indentation (⌃ + I)

Another useful keyboard shortcut to help you keep your code clean and neatly formatted. This is especially handy given how quickly indentation in SwiftUI can get out of control.

![image](/assets/images/xcode-kb-shortcut-balance-indentation.gif)

## Navigation shortcuts

Knowing how to navigate effectively within Xcode can help you switch contexts quickly. It can also help you find what you looking for without manually searching for it.

### Quickly open (⇧ + ⌘ + O)

This keyboard shortcut is the one I use most frequently, all the time. Instead of manually navigating and searching for a file in the project navigation you can use the open quickly keyboard shortcut.

It works great for just about anything you want to navigate to, including files, types, functions, etc.

![image](/assets/images/xcode-kb-shortcut-open-quickly.gif)

### Jump to definition (⌃ + ⌘ + J)

Another keyboard shortcut I use often is to jump to the definition of a type, function, etc. Simply set your cursor on, for example, the type or function that you what to jump to and press `⌃ + ⌘ + J`.

![image](/assets/images/xcode-kb-shortcut-jump-to-definition.gif)

## Views shortcuts

Additional views like the project navigator, inspectors, preview, and debug area can be useful but not always needed. Often times they take up a lot of screen real estate when not needed, especially when you're working on a smaller screen. Knowing how to hide and show these views as needed can free up a lot of screen space for the editor, allowing you to see more of your code.

### Toggle canvas / preview  (⌥ + ⌘ + ↩)

This keyboard shortcut is extremely useful when working with SwiftUI when you want to show/hide the preview.

![image](/assets/images/xcode-kb-shortcut-toggle-canvas.gif)

### Open library (⇧ + ⌘ + L)

Another frequent action is to open the library to insert code snippets, images, add view modifiers, etc.

Using the library is also a great way to see what modifiers are available for a given view in SwiftUI.

![image](/assets/images/xcode-kb-shortcut-open-library.gif)

## Cheatsheet

Below is a table and overview of some of the most useful and frequently used keyboard shortcuts in Xcode:

> - ⌘ = Command
> - ⌥ = Option/Alt
> - ⇧ = Shift
> - ⌃ = Control
> - ←→ ↑↓ = Arrow keys
> - ↩ = Enter
{: .prompt-info }

| Category   | Command                                | Shortcut      |
| ---------- | -------------------------------------- | ------------- |
| Editor     | Go to beginning of line                | ⌘ + ←         |
| Editor     | Go to end of line                      | ⌘ + →         |
| Editor     | Move line up                           | ⌥ + ⌘ + [     |
| Editor     | Move line down                         | ⌥ + ⌘ + ]     |
| Editor     | Code completion                        | ⌃ + Space     |
| Editor     | Comment line / selection               | ⌘ + /         |
| Editor     | Balance indentation                    | ⌃ + I         |
| Editor     | Delete line                            | ⌘ + D         |
| Editor     | Open library                           | ⇧ + ⌘ + L     |
| Editor     | Generate documentation                 | ⌥ + ⌘ + /     |
| Navigation | Open quickly                           | ⇧ + ⌘ + O     |
| Navigation | Switch tabs left                       | ⇧ + ⌘ + [     |
| Navigation | Switch tabs right                      | ⇧ + ⌘ + ]     |
| Navigation | Show current file in project navigator | ⇧ + ⌘ + J     |
| Navigation | Go forward in navigation history       | ⌃ + ⌘ + →     |
| Navigation | Go back in navigation history          | ⌃ + ⌘ + ←     |
| Navigation | Jump to definition                     | ⌃ + ⌘ + J     |
| Navigation | Go to line                             | ⌘ + L         |
| Navigation | Switch between code editors            | ⌘ + J         |
| Navigation | Jump bar                               | ⌃ + 1 - 6     |
| Navigation | Find selected symbol                   | ⇧ + ⌃ + ⌘ + F |
| Navigation | Find call hierarchy                    | ⇧ + ⌃ + ⌘ + H |
| Views      | Toggle canvas / preview                | ⌥ + ⌘ + ↩     |
| Views      | Toggle assistant                       | ⌃ + ⌥ + ⌘ + ↩ |
| Views      | Reload preview                         | ⌥ + ⌘ + P     |
| Views      | Toggle project navigator               | ⌘ + 0         |
| Views      | Toggle inspectors                      | ⌘ + ⌥ + 0     |
| Views      | Toggle debug area                      | ⇧ + ⌘ + Y     |
| Debugging  | Toggle breakpoint                      | ⌘ + \         |
| Debugging  | Step over                              | F6            |
| Debugging  | Step into                              | F7            |
| Debugging  | Step out                               | F8            |
| Debugging  | Continue to current line               | ⌃ + ⌘ + C     |
| Debugging  | Continue                               | ⌃ + ⌘ + Y     |

## Conclusion

Mastering these keyboard shortcuts might seem a bit overwhelming at first. But once you've got then down it almost becomes natural and you'll find yourself using them without thinking about it.

Great for keeping both your hands on the keyboard more often making you a lean, mean code typing machine!
