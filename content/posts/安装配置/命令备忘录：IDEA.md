---
title: 命令备忘录：IDEA
date: 2021-04-02 09:50:12
categories:
- 安装配置
- 应用软件
tags: 
- Linux
---

## Editor Basics

### Actions

`Ctrl + Shift + A`

### Select

`Ctrl + Shift + →`	Select a word

`Ctrl + Shift + ←`	Select a word

`Ctrl + W`	Extend the select

`Ctrl + Shift + W`	Shrink the select

`Ctrl + A `	Select whole file

### Comment Line

`Ctrl + /`	Comment out or uncomment line(s)

### Delete Line

`Ctrl + Y`	Delete line(s)

`Ctrl + Z` 	Restore the deleted line(s)

### Duplicate

`Ctrl + D` 	Duplicate line(s)

### Move

`Alt + Shift + ↓`	pull down line(s)

`Alt + Shift + ↑`	pull up line(s)

`Ctrl + Shift + ↓`	pull down class member

`Ctrl + Shift + ↑`	pull up class member

### Collapse

`Ctrl + -`	collapse a region(eg. method)

`Ctrl + +`	expand a region

`Ctrl + Shift + -`	collapse all regions

`Ctrl + Shfit + +`	expand all regions

### Multiple Selections

`Alt + J`	Select the symbol at the caret or select the next occurrence of this symbol

`Alt + Shift + J`	deselect the last occurrence

`Ctrl + Alt + Shfit + J`	select all occurrence in the file

## Code Completion

### Basic Completion

`Ctrl + SpaceKey `	activate basic completion(replace `Enter` with `tab` to overwrite the word rather than simply insert it)

### Smart Type Completion

`Ctrl + Shift + SpaceKey `	activate smart type completion

### Statement Completion

`Ctrl + Shift + Enter`	complete the statement

## Refactorings

### Rename

`Shift + F6`	rename

### Extract Variable/Field

`Alt + Shift + V`	introduce local variable or field(you can also use `Alt + Enter` and select the list item )

### Refactoring Basics

`Ctrl + Alt + C`	extract the selected expression into a constant

`Ctrl + Alt + M`	extract the selected code block into a method

`Ctrl + Alt + P`	extract the selected expression into a parameter (*not work*)

## Code Assistance

### Code Formatting

`Alt + Shift + L`	correct code formatting

### Parameter Info

`Ctrl + P` see the method signature

### Quick Popups

`Ctrl + Q`	see documentation  for the symbol at the caret(`Esc` to close the popup)

`Ctrl + Shift + I`	see the definition of the symbol at the caret

### Editor Coding Assistance

`F2`	go to the next highlighted error in the file

`Ctrl + 1`	see the error description

`Alt + Enter`	show suggestions and fix it

`Ctrl + Alt + T`	surround with

`Ctrl + Shift + 7`	highlight all usages of the symbol at the caret

## Navigation

### Jump to source

`F4`	jump to source(include jdk classes)

###  Declaration/Implementation

`Ctrl + B`	jump to the declaration of a class or interface

`Ctrl + Alt + B`	see implementations of a class/interface

### File Structure

`Ctrl + 0`	see file structure

### Next/Previous Occurrences

`Ctrl + F`	find all word  you selected

`F3`	find next occurrences

`Shfit + F3`	find previous occurrences
