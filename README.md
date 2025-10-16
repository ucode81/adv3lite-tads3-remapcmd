# adv3lite-tads3-remapcmd v1.0
Added a true RemapCmd class that takes generic text.

## Background
One of the complaints that a number of adv3lite/adv users have is the Doer.  While the Doer is a solid way for redirecting or remapping commands, it has one major downside: it does not take arbitrary text and has limited matching capability.  What is needed is something that allows for more flexible input.

## Specification
What do we want in our RemapCmd
- Source phrase used for matching can be arbitrary text in that the words used do not need to be any known object (noun, verb, preposition, etc.)
- Source phrase can have variations using or ("|"), grouping with "(...)" and multiple phrases with ";"
- When the source phrase matches, one should have several options for how this gets processed
  - Provide a complete phrase to replace it (this one, though, MUST use words understood by the game)
  - OR emit some set of messages to the console
  - OR redirect to some command
- In addition, the specified remap command could optionally be
  - location-sensitive
  - have a when condition
  - have a during condition (for Scenes)

 (Thanks for Eric Eve for those last two suggestions.)

## RemapCmd Documentation
### Template
RemapCmd via the template is easiest way to use this.  In your main header file, add the following template:

`RemapCmd template 'cmd' @where? 'remappedCmd'?;`

### Usage
RemapCmd has one source text 'cmd' that contains one or more phrases (separated by ';') that supports grouping via '(...)' and or via '|'.  Here is an example (and, yes, most of this would work natively but this is just an example).

We have a verb HoldUp (hold up) that takes one object.  However, *light* and *ceiling* are not nouns and we don't want to define them but want to handle the paper.  When you write your source text, it is very similar to VerbRule except you do not need any '...' (although they are allowed but must be escaped inside of an existing '...' string).

`RemapCmd 'hold paper up to light;hold paper against (|ceiling) light' @brightRoom 'hold up paper';`

So, how this works is that RemapCmd will look at the following phrases as a match:
- `hold paper up to light`
- `hold paper against light`
- `hold paper against ceiling light`

If this matches, RemapCmd will replace that string with the command `hold up paper` -- as if that is what you typed in.  However, it will ONLY do this if the current location is `brightRoom`.  (The *where* location can be a Room, Region, or a list of Rooms and/or Regions.)

Another use case is to just emit some message(s) to the console. This is a great way to respond to some text without having to define all (or any) of the objects which entails running some additional code. You do this by **NOT** specifying the remapped text and defining  `execute()` function inside of RemapCmd:

```
RemapCmd 'spit (|(on the ground))'
  execute()
  {
    "How often have you told yourself to quit doing that... ";
  }
;
```
So now when you run this, here is what you see:
```
> spit on the ground
How often have you told yourself to quit doing that...

>
```

The addition of `execute()` opens up other possibilities.  Let's look at another example from the Colossus Cave (thanks to Eric Eve for the suggestion):

```
RemapCmd 'fee fie foe foo' @giantRoom
  execute()
  {
    eggs.moveInto(giantRoom);
    "The eggs {i} gave to the troll have mysteriously re-appeared on the floor in front of {me}. ";
  }
  when = eggs.isIn(troll)
;
```
---
**NOTE:** RemapCmd has a restriction on the source text phrase in that, outside of its the special characters described above, it does not allow any other special or punctuation characters other than comma ',' or apostrophe (').

### Operation
RemapCmd parses up the cmd and does not make any attempt to map the words to existing objects in the game.  Rather, it builds all the possible variations (by the use of "|", "(" and ")" operators) and then matches that against the user input FIRST before any other routines get a whack at it.  You can also provide two or more disjoint phrases that are separated by semi-colon ";" to provide additional phrases that match.

Phrases can contain words offset with single quotes ('..') but you will need to use backslash as in `\'` within the text itself. This capability can be used to have a compound word with a space in it treated as one effective word.

RemapCmd takes advantage of string hashing to ensure high performance within your game.

It does modify the `parse(str)` command in `parser.t`.

## Installation/setup
- Include `remapcmd.t` in your project
  
Please provide feedback if you have any questions or suggestions.
