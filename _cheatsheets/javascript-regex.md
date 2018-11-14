---
layout: cheatsheet
title:  "Javascript Regex Cheat Sheet"
categories: emoji
tags: [cheetsheet emoji]
---
## Table of Contents
- [Basics](#basics)
- [Character Classes](#character-classes)
- [Flags](#flags)
- [Quantifiers](#quantifiers)
- [Assertions](#assertions)
- [Special Characters](#special-characters)
- [Groups](#groups)
- [Replacement](#replacement)

## Basics

| --- | --- |
| . | Any character except newline |
| a | The character a |
| ab | The string ab |
| a\|b | a or b |
| a\* | 0 or more a's |
| \ | Escapes a special character |

## Character Classes

| --- | --- |
| [ab-d] | One character of a, b, c, d |
| [^ab-d] | One character except a, b, c, d |
| [\b] | Backspace character |
| \d | One digit |
| \D | One non-digit |
| \s | One whitespace |
| \S | One non-whitespace |
| \w | One word character |
| \w | One non-word character |

## Flags

| --- | --- |
| g | Global match |
| i | Ignore case |
| m | ^ and $ match start and end of line |

## Quantifiers

| --- | --- |
| * | 0 or more |
| + | 1 or more |
| ? | 0 or 1 |
| {2} | Exactly 2 |
| {2,5} | Between 2 and 5 |
| {2,5} | Between 2 and 5 |

## Assertions

| --- | --- |
| ^ | Start of string |
| $ | End of string |
| \b | Word boundary |
| \B | Non-word boundary |
| (?=..) | Positive look ahead |
| (?!..) | Negative look ahead |

## Special Characters

| --- | --- |
| \n | New line |
| \r | Carriage return |
| \t | Tab |
| \0 | Null Character |
| \YYY | Octal character YYY |
| \xYY | Hexadecimal character YY |
| \uYYYY | Hexadecimal character YYYY |
| \cY | Control character Y |

## Groups

| --- | --- |
| (..) | Capturing group |
| (?:..) | Non-capturing group |
| \Y | Match the Y'th captured group |

## Replacement

| --- | --- |
| $$ | Insert $ |
| $& | Insert entire match |
| $` | Insert preceding string |
| $' | Insert following string |
| $Y | Insert Y'th captured group |
