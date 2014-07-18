---
---

# Marsyas Scripting Language

The Marsyas scripting language is a declarative language to specify the structure and behavior of a network of MarSystems.

Script files can be used by various programs within the MarSystem framework, for example the GUI tool 'Inspector', or the command-line tool 'marsyas-run', as well as parsed into a MarSystem using the Marsyas C++ library API.

**Note:** Spaces and line breaks never have any significance, so in this regard, the following examples can be freely reformatted to your liking.

## The Script File

The expected filename extension is ".mrs".

The script as a whole is expected to be a definition of a single MarSystem containing a complete network hierarchy:

**Syntax:**

> *script-file* ::= *marsystem*

## MarSystem

**Syntax:**

> *marsystem* ::= [ *marsystem-name* __":"__ ] *marsystem-type* [ __"{"__ *marsystem-body* __"}"__ ]  
> *marsystem-body* ::= * ( *control-declaration* | *child-declaration* | *marsystem-prototype* )

To declare an instance of a MarSystem, type the desired instance name followed by a colon `:` followed by the name of a MarSystem type:

    sine: SineSource

The instance name is optional; however, it will allow you to refer to that instance from elsewhere in the script.

The optional body of the MarSystem within the curly braces `{ }` will contain declaration of control values, child MarSystems, and MarSystem prototypes, as described in the continuation of this document.


    sine: SineSource { }


## Child MarSystems

**Syntax:**

> *child-declaration* ::= __"->"__ *marsystem*

To declare children of a composite MarSystem, simply type the arrow symbol `->` followed by the complete declaration of a MarSystem as defined above, all within the curly braces of the parent MarSystem. Repeat the same procedure for further children.

For example, a Series containing a SineSource followed by a Gain:

    Series { -> SineSource -> Gain }


A more elaborate example of a Series containing two named SineSources in parallel, followed by a named Gain.

    Series {
        -> Parallel {
            -> sine1: SineSource
            -> sine2: SineSource
        }
        -> volume: Gain
    }


## Control Values

**Syntax:**

> *control-declaration* ::= [ __"+"__ ] [ __"(public)"__ ] *control-name* __"="__ *control-value*  
> *control-value* ::= *literal-value* | *control-path* | *expression-value*

To declare control values for a MarSystem, add a sequence of control assignments to the body of its declaration.

For example, to declare the frequency of a SineSource to be a constant value:

    SineSource { frequency = 432.0 }


To declare a SoundFileSource to read from a particular file, starting at a particular position (in samples):

    SoundFileSource
    {
        filename = "path/to/my-awesome-music.wav"
        pos = 128000
    }

Note that there is no requirement to declare any control. All undeclared controls will take on their default values.

If a plus sign `+` is prepended to a control declaration, a new control with the given name and value will be created. This is especially useful in combination with control links, expression values and prototypes (see explanation and examples below).

Moreover, if the keyword `(public)` (including the parenthesis) is prepended to the control name, the control will get a special "public" status. This currently doesn't affect MarSystem behavior in any way, but it allows Marsyas tools (for example the Inspector) to hide the other (non-public) controls wich are not meant for direct interaction by the user.


### Literal Values

**Syntax:**

> *literal-value* ::= *boolean* | *integer* | *real* | *realvec* | *string*  
> *boolean* ::= __"true"__ | __"false"__  
> *realvec* ::= __"["__ *realvec-row* \*( __";"__ *realvec-row*  ) __"]"__  
> *realvec-row* ::= *real* \*( __","__ *real* )  
> *string* ::= __'"'__ *any-character* __'"'__

Examples of all kinds of literal control values:

    boolean = true
    integer = 456
    real = 456.78
    realvec = [ 11, 12, 13; 21, 22, 23 ]
    string = "path/to/my-awesome-music.wav"

**Note:** Since each control has an inherent type, the values must match the type of the control, or else the assignment will fail.


### Control Links

**Syntax:**

> *control-path* ::= [ *marsystem-path* ] *control-name*  
> *marsystem-path* ::= [ __"/"__ ] *( marsystem-name __"/"__ )

A control can be linked to another control of the same MarSystem or any other Marsystem within the network. That means that whenever the value of any of the linked controls is changed, the other's value will change accordingly.

A control link is declared using a path to another control on the right hand side of the assignment. The path of a control is a combination of the path of a MarSystem within the network, and the name of one of its controls to link to. The path of a MarSystem is a sequence of MarSystem names separated by slashes `/`, where each previous MarSystem is the next one's parent. However, the topmost MarSystem's name is never used - it is represented simply with a slash `/` at the beginning of the path. If the path begins with a slash, it is an absolute path, otherwise it is interpreted relative to the MarSystem where it is used.

For example: the second SineSource will always have the same frequency as the first one:

    Parallel
    {
        -> sine1: SineSource { frequency = 600.0 }
        -> sine2: SineSource { frequency = /sine1/frequency }
    }


The following example uses a control link to expose the 'filename' control of a SoundFileSource as a public top-level control (initialized to an empty string):

    Series
    {
        + (public) input = ""
        -> SoundFileSource { filename = /input }
    }

In fact, it doesn't matter much whether control A is linked to control B, or the other way around. The direction only affects which one's value before linking will be applied to both immediately after linking.



### Expression Values

**Syntax:**

> *expression-value* ::= __"("__ *expression* __")"__  
> *expression* ::= *operation* | ( __"("__ *operation* __")"__ )  
> *operation* ::= *operand* \*( *operator* *operand* )  
> *operand* ::= *literal-value* | *control-path* | *expression*  
> *operator* ::= __"+"__ | __"-"__ | __"*"__ | __"/"__ | __"=="__ | __"!="__ | __"<="__ | __">="__

A value of a control can be an expression: a combination of literal values, links to other controls and operators between them. Operators comprise the common arithmetic operators (`+ - * /`) and comparison operators (`== != <= >=`).

If an expression contains links to other controls, it will be re-evaluated whenever those controls change, and a new resulting value will be applied to the control to which the expression is assigned.

For example: this Gain will attenuate the output of the SineSource in relation to its frequency:

    Series {
        -> sine: SineSource { frequency = 600.0 }
        -> Gain { gain = (100.0 / /sine/frequency) }
    }


The following example creates a series of harmonic oscillations at multiples of the fundamental frequency provided as an additional public top-level control:

    Parallel
    {
        + (public) frequency = 330.0
        -> SineSource { frequency = /frequency }
        -> SineSource { frequency = (/frequency * 3.0) }
        -> SineSource { frequency = (/frequency * 5.0) }
        -> SineSource { frequency = (/frequency * 6.0) }
    }


**Note:** As of this writing, the following limitations apply:

- Only operands of same type are allowed in an operation.
- Addition, subtraction, multiplication and division can only be performed with integer and real values.


## Prototype MarSystems

**Syntax:**

> *marsystem-prototype* ::= __"~"__ *marsystem-name* __":"__ ( *marsystem* | *filename* )  
> *filename* ::= *string*

A prototype is a MarSystem network that itself becomes a MarSystem type. Whenever it is instantiated, a complete copy of the prototype network is created, and additional children and control assignments declared for the particular instance will be merged in.

### Inline Prototypes

A prototype may be declared inline, anywhere within a network. The entire syntax is almost the same as for declaring a child, except that the tilde `~` prefix is used instead of the arrow `->` prefix. In addition, the prototype must be given a name, which will become a new type-name.

Another important distinction with a child declaration is that a prototype declaration defines its own path scope. No control path within the prototype can relate to something outside the prototype; absolute paths are interpreted relative to the top-most MarSystem of the prototype.

The following example declares a Channel prototype that composes a SineSource and a Gain, and then instantiates 3 such Channels in parallel, with individual control values. The prototype internally uses control links to expose the useful controls as its own top-level controls, for ease of access.

    Parallel
    {
        ~ Channel : Series
        {
          + frequency = 440.0
          + gain = 1.0
          -> SineSource { frequency = /frequency }
          -> Gain { gain = /gain }
        }

        -> Channel { frequency = 200.0  gain = 1.0 }
        -> Channel { frequency = 400.0  gain = 0.2 }
        -> Channel { frequency = 600.0  gain = 0.5 }
    }


### External Prototypes

A prototype may also be declared as the network defined in another script file. In that case, the type-name part of the prototype declaration is simply replaced with a string containing the path to the script file. Relative paths are interpreted in relation to the current script file where they are used.

For example, consider that the above-introduced Channel network was defined in a file of its own:

File *channel.mrs:*

    Series
    {
        + frequency = 440.0
        + gain = 1.0
        -> SineSource { frequency = /frequency }
        -> Gain { gain = /gain }
    }


Then another script file could use it as a prototype like this:

    Parallel
    {
        ~ Channel : "channel.mrs"
        -> Channel { frequency = 200.0  gain = 1.0 }
        -> Channel { frequency = 400.0  gain = 0.2 }
        -> Channel { frequency = 600.0  gain = 0.5 }
    }
