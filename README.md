# adv3lite-tads3-remapcmd
Added a true RemapCmd class that takes generic text

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
- In addition, the specified remap command could optionally be location-sensitive

## RemapCmd Documentation
### Template
RemapCmd via the template is easiest way to use this.  In your main header file, add the following template:
`RemapCmd template 'cmd' @location? 'remappedCmd'?;

### Usage
RemapCmd has one source text 'cmd' that contains one or more phrases (separated by ';') that supports grouping via '(...)' and or via '|'.  Here is an example (and, yes, most of this would work natively but this is an example, starting with a simple one:

We have a verb HoldUp (hold up) that takes one object.  The light is not known and we don't want to define it, so we write:

`RemapCmd 'hold paper up to light;hold paper against (\'\'|ceiling) light' @brightRoom 'hold up paper';`

(RemapCmd accepts words enclosed by '...' but you have to escape them in source text; this is the only way to include an empty-string word.)

So, how this works is that RemapCmd will look at the following phrases as a match:
- hold paper up to light
- hold paper against light
- hold paper against ceiling light

If this matches, RemapCmd will replace that string with the command 'hold up paper' -- as if that is what you typed in.  However, it will ONLY do this if the current location is `brightRoom`.

Less common usage would be to execute some code.  You do this by NOT specifying the remapped text and defining an `execute()':
```
RemapCmd 'hold paper up to light;hold paper against (\'\'|ceiling) light' @brightRoom
  execute()
  {
    doInstead(HoldUp,paper);
  }
;
```

While `execute()` allows one to write arbitrary code, you should limit what you do.  A third use case is to just emit some message(s) to the console.  This is a great way to respond to some text without having to define all of the objects.  Say we have included rocks in our description of some location, but do not have a rock object:

```
RemapCmd '(throw|toss) rock ((at sky)|(up in the air))'
  execute()
  {
    "(first picking up a rock)\nYou throw the rock high in the sky and it comes back down with a plop. ";
  }
;
```
So now when you run this, here is what you see:
```
> toss rock up in the air
(first picking up a rock)
You throw the rock high in the sky and it comes back down with a plop. 

>
```

### Operation
RemapCmd parses up the cmd and does not make any attempt to map the words to existing objects in the game.  Rather, it builds all the possible variations (by the use of "|", "(" and ")" operators) and then matches that against the user input FIRST before any other routines get a whack at it.  You can also provide two or more disjoint phrases that are separated by semi-colon ";" to provide additional phrases that match.

Phrases can contain words with offset by '..' but you will need to use backslash as in `\'`; this can be used to have a compound word with a space treated as one object and is the only way to add an empty string for some match condition.

RemapCmd takes advantage of string hashing to ensure high performance within your game.

It does modify the `parse(str)` command in `parser.t`.

## Installation/setup
- Include `remapcmd.t` in your project
  
Please provide feedback if you have any questions or suggestions.
