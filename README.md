*This is not a complete documentation! While it does cover Quanta's existing features, it is missing what I have planned for the language!*
*See also: [Change log](https://github.com/aadenboy/Quanta/blob/main/changelog.md)*

![Quanta logo](https://github.com/aadenboy/Quanta/blob/main/QuantaLogoFull.png?raw=true)
Quanta is a human-first data representation language inspired by [JSON](https://en.wikipedia.org/wiki/JSON) and [YAML](https://en.wikipedia.org/wiki/YAML) with the intent of being easy to read and write while also being expressive enough to handle a multitude of data structures and domains.

Sections marked with a warning symbol (⚠) cover features which may be more complex compared to what you need, and can be safely ignored for most use cases.

## Etymology
The name "Quanta" comes from [quantum mechanics](https://en.wikipedia.org/wiki/Quantum_mechanics), which is often attributed to atoms. The objects in Quanta are seen as the nuclei of atoms, and their properties are their electrons, with all the atoms bonding together to create a molecule. The name is also partially to prevent the language's name from being an acronym, unlike many of the commonly used data formats.

## Syntax
It is important to note that Quanta is designed to minimize any possible errors. The syntax is intended to allow you to be as expressive with it as you need, while also disallowing Quanta from making incorrect assumptions. In addition to this, the structure of a file should be easy to grasp, with the addition of freeform placement.

## Structure
At the base level, you have objects and attributes. Objects are the foundation of any Quanta file, and the attributes are what compose the objects. In a way, the Quanta file itself is its own object, containing several more objects. Object names are not unique identifiers (see [IDs](#IDs)). Instead, they're more like datatypes, or class names. Depending on your usage, you can treat objects as individual sections of a log, or as the building blocks of a world.

### Objects
You define an object by wrapping its name/type inside square brackets. Anything that comes after this point is a part of the object, until another object is defined.
```
-- unique namess
[object 1]
[object 2]
[object 3]
[object 4]

-- or same names
[type A]
[type A]
[type B]
[type B]
```

How exactly the parser stores these objects is explained later.

### Attributes
The attributes of an object are defined after the object, by placing the name of the attribute inside curly braces.
```
[object]
  {attrA} 1 -- same as object.attrA = 1
  {attrB} 2
  {attrC} 3
  
[object]
  {attrA} 5 -- note that this is not the same attrA as the previous object
  {attrB} 7 -- so we can use different values here without overwriting
  {attrC} 9
  
-- remember, the order and spacing of attributes is entirely up to you, so long as they follow an object
[object] {attrA} 2 {attrB} 3 {attrC} 5
```

### IDs
If a unique identifier for an object is necessary, you can either do so by setting an attribute, or by placing a [word](#Words) after the object's definition. Under convention, you should start the identifier with an exclamation mark.
```
[object] !id1
  {value} 25
  
[object] !id2
  {value} 13
  
[object] !id3
  {name} 7
```

### Lists
You can create a list simply by separating each value within the list with a space.
```
[object]
  {list1} 1 2 3   -- same as object.list1 = [1, 2, 3]
  {list2} 1, 2, 3 -- do NOT use commas
```

⚠ In the case that more granularity is needed, you can index a specific position in the list with a hashtag.
```
[object]
  {list} 1 2 3 -- object.list = [1, 2, 3]
  {list#4} 4   -- object.list[4] = 4
  {list#1} 5   -- object.list[1] = 1
  
[object]
  {list#1} 1     -- since list is empty, Quanta creates an empty list for you
  {list#2} 1 2 3 -- you can nest lists inside lists this way; object.list = [1, [1, 2, 3]]
```
To maintain readability, you should only use this syntax to insert deeper levels. You should also consider a directory format instead (see below).

### Directories
Directories are created in a similar manner by separating each level with a colon.
```
[object]
  {attr1:foo} 1 -- since attr1 is empty, Quanta automatically makes it a directory; object.attr1.foo = 1
  {attr1:bar} 2 -- object.attr1 = {foo = 1, bar = 2}
  {attr2:dir1:dir2:value} 3 -- object.attr2.dir1.dir2.value = 3
```
For cleaner code, an alternate syntax is available. Ending an attribute with a colon turns it into an empty directory. Empty directories can be referenced by beginning an attribute with a colon, which automatically expands it into the full path. Starting with two colons steps down a level.
```
[object]
  {dir:}     -- creates an empty directory
    {:foo} 3 -- same as {dir:foo}
    {:bar} 4 -- you can do this as many times as you need
    {:baz:}  -- and nest deeper
      {:sub} 5 -- {dir:baz:sub} 5
    {::higher} 6 -- {dir:higher}
```
To maintain readability, try to minimize the amount of upward traversals (`::`) you use.

## Types
There are eight types available in a Quanta file: words, strings, numbers, booleans, none, colors, aliases, and flags.

### Words
Words are the most basic type, acting identically to a non-quoted string. These are the default—if a sequence of characters cannot correspond to another data type, it is a word.
```
[speaker]
  {volume} 50
  {channels} stereo -- this is a word
  {name} st_5       -- this is also a word
  {modes} phaser reverb
```

In a way, words are a graceful fallback for any bad values.
```
[errors]
  {malformed} 1-1 ++2 1.3.5
  {keymash} oaipguiosdjf r\]elpwqe 01=-0sds-fdsf
  {mistake} "not a string" -- same as ::"not:: ::a:: ::string"::
```

Words are best used for, as they're named, words.

### Strings
Strings allow you to include more than one word or special characters into a single value. They provide more functionality than words, and under the surface have several features available. Unlike strings in other languages, Quanta delimits strings using a double-colon, an uncommon occurence unlike quotes. Strings are implicitly multi-lined.
```
[strings]
  {simple} ::Hello, world!::
  {dialogue} ::And he said, "I CAN TALK!" without the use of any backslashes.::
  {empty} ::::
  {colon} ::You can also use colons: like that.::
  {haiku} ::This is a long string
No \n is needed
Just press down enter::
```

#### Escape codes
Several escape codes are provided. Additionally, any character can be escaped without harm.
| Code | Character |
|---|---|
| `\\` | A single backslash |
| `\:` | A colon (allows `\::`) |
| `\0` | Null character (U+0000) |
| `\n` | Newline (U+000A) |
| `	` | Tab (U+0009) |
| `\r` | Carriage return (U+000D) |
| `\b` | Backspace (U+0008) |
| `\f` | Form feed (U+000C) |
| `\v` | Vertical tab (U+000B) |
| `\a` | Alert/Bell (U+0007) |
| `\e` | Escape (U+001B) |
| `\u{code}` | UTF-8 character corresponding to `code` |
| `\xXX` | ASCII character corresponding to `XX`, which are two hexadecimal numbers |
| `\cC` | Escape character corresponding to the code of `C` modulo 32, i.e `\cJ` is equivalent to `\n` (72 mod 32 = 10) |

#### Block comments
You can create a block comment by containing the newlines inside of a string.
```
[example]
  {attr} data   -- this is a really long comment ::
                :: which explains this attribute ::
                :: see how I use strings here?   :: <-- string start
 string end --> :: they contain the ending newline
  {attr2} data
  {attr3} new   --::
                    you can also do it like this
                    so long as you close it afterwards
                ::
```
For readability, try to stick to one style and not treat them like strings.

## Numbers
Numbers can be naturally defined in multiple ways. All forms of integers and floats are supported.

```
[integers]
  {natural} 1 2 3 4 5
  {reals} 0 -1 100 001
  {floats} 0.5 .5 -0.5 -.5
  {scientific} 1e7 -0.5e3 5e-2
```

Quanta also supplies several bases.

| Base | Prefix | Characters |
|---|---|---|
| Hexadecimal | `0x` | `0123456789abcdef` |
| Binary | `0b` | `01` |
| Trinary | `0t` | `012` |
| Octal | `0o` | `012345678` |

```
[base]
  {hex} 0xff 0x9a
  {bin} 0b11111110 0b10011010
  {tri} 0t11111110 0t12201
  {oct} 0o376 0o232
```

Floats in other bases are also supported.

```
[floats]
  {hex} 0x1e.5b
  {bin} 0b11110.01011011
  {tri} 0t1010.10012101020020000022
  {oct} 0o36.266
```

## Booleans and `none`
The booleans `true`, `false`, and `none` are the only reserved keywords used by Quanta. The latter equals null or nil in common programming languages.

```
[incomplete]
  {stats} true
  {debug} false
  {flags} mode1 mode2 none none none mode6 -- indexes 3 to 5 are empty
  {flags#1} none -- unset flags[1]
  {flags#4} mode4 -- set flags[4]
```

## ⚠ Aliases
Aliases are the first special type. They can be defined in the data or by the program, and function like variables in common programming languages. To define an alias, use the `@alias` flag, followed by the name, then value. The same traversal rules apply here. Note that lists and objects defined as an alias should point to the same list/object every time it is used. To refer to an alias, supply the name of the alias between a dollar sign and colon.

```
@alias hello ::Hello, world!::
@alias favnum 4
@alias primary #f00 #0f0 #00f
@alias object:
  @alias :prop1 true
  @alias :prop2 false
  
-- same string each time
[program] !prog1 {output} $hello;
[program] !prog2 {output} $hello;
[program] !prog3 {output} $hello;

[misc] {value} $favnum; $favnum;

[palette]
  {base} $primary;
  {other} $primary;
  {other#4} #ff0
  {other#5} #f0f
  {other#6} #0ff

-- complex structures may be easier to compose this way
[complex] {datafield} $object; $object; $object;
```

## Flags
Flags are the second special type. While they don't correspond to an actual value, they are used to tell the parser what to do. In addition to this, a program can define their own flags. Currently, Quanta only has two built-in flags:

### ⚠ `<holds>`
The `<holds>` flag defines the value of the attribute to correspond to the next defined object. Inserting the `<end>` flag at the end of any attribute will immediately return the scope to the parent object.

```
[object]
  {basedata} 1 2 3
  {morebase} true false
  {child} <holds>
    [child]
      {data} lorem ipsum dolor sit amet
      {moredata} 1 2 3 4 5
      <end>
```

### ⚠ `<container>`
The `<container>` flag defines a container which holds more objects in a list. It should be placed into a value, and followed by a word to use as the syntax marker for an entry to the container. Objects which are added to the list should be preceded by the word. A container is closed by marking the next entry with the `<end>` flag instead, immediately returning the scope to the object which holds the container.

```
[object]
  {basedata} 1 2 3
  {morebase} true false
  {moreobjects} <container> * -- this tells Quanta to use an asterisk as an entry marker, but any word will do
    * [object2] {data} lorem ipsum
    * [object2] {data} dolor sit amet
    * [object2] {data} ::continue forever::
    * <end> -- since Quanta cannot infer when the container ends, you must tell it to directly
  {extradata} 5 6 7
```

Quanta allows for multiple containers to be open at the same time, and will append each object to whichever container uses the appropriate prefix. Note that containers will not automatically close, as Quanta is unable to infer when this happens without the marker.

## Example
For an actual example, take the data used by the file [File:Mover.gif](https://cellua.miraheze.org/wiki/File:Mover.gif) on the CelLua Machine Wiki:

```
[board]
  {level} ::K3\::;1g;c;1;eNozMBgFJAALGCM9PR1MBwYGAslsRzATSGgY5sAVB46G1ygYBVQGAGJF41U=;::
  {camera} 1 1
  {cellsize} 40
  {capture} 11 7

[animation]
  {defaultspeed} 0.2
  {mode} ticks
  {cellposition} aliased
  {fps} 30
  {ticks}  0 -> 1 -> 22 -> 23 -> 24 -> 25 -> 26 -> 27 -> 28 -> 29 -> 36 -> 39    -> 45 -> 48   -> 53
  {camera} 0 -> 1 -> 21 -> 0  -> 1  -> 0  -> 1  -> 0  -> 1  -> 0  -> 7  -> -1+2i -> 6  -> 7-2i -> 5
```

The first line `[board]` defines a board object. In the case of the GIF, this essentially defines the `board` section of the entry. The next line, `{level}`, sets the attribute `block.level` equal to the string after it. Note the use of a backslash behind the double-colon in the string. Afterwards, the attributes `{camera}`, `{cellsize}`, and `{capture}` are defined as a number list, number, and one final number list.

After that is `[animation]`, which defines the `animation` section of the entry. `{mode}` and `{cellposition}` use words to define their values rather than another value. For their purpose, they're short, meaningful values that the program can easily parse, much like an enum.

At the final part, `{ticks}` and `{camera}` each contain a long list of values (note the variable spacing). In the case of `{ticks}`, it's a neat list of a number-word pattern. For `{camera}`, the pattern isn't as clear, as you may notice the inclusion of two words `-1+2i` and `7-2i` at the end (since they aren't real numbers). The consumer program will parse these independently of Quanta.

