---
---

# Architectural overview

Marsyas is a framework for composition and coordinated execution of data-processing modules. It fits into the *actor* or *dataflow* programming paradigm. Aside from facilitating composition of modules, Marsyas also provides a huge collection of particular modules that do different kinds of processing, mainly for the purpose of Music (Information) Retrieval tasks.

## MarSystems

The fundamental element of the framework is the abstract type MarSystem which represents an individual processing element. All concrete processing elements are subtypes of MarSystem.

A MarSystem takes as input a 2-dimensional matrix of real numbers and produces output data of the same kind. Different columns of the matrix represent successive moments in time, while different rows represent data that occurs at the same moment in time. Hence, columns are called *samples* and rows are called *observations*. In addition, a third attribute is associated with input and output: *sample rate*. It describes the rate of succession of samples in time and some processing depends on it.

Input and output of a MarSystem can be of different formats (numbers of rows and columns and sample rate). The specific kind of processing of a MarSystem determines the dependency between the input and output formats. Typically, a MarSystem can work with any input format and will adjust its output format accordingly. For example: the `Spectrum` MarSystem which performs a DFT on the input data will accept a row of samples of any size as input and will produce a column of equal number of elements as output. The output sample rate will be the division of the input sample rate with the number of input samples.

The abstract MarSystem type also facilitates hierarchical composition of modules: each MarSystem can have other MarSystems as children. The purpose of this is for the parent MarSystem to coordinate execution and data passing between its child MarSystems. Concrete MarSystem subtypes that perform this kind of tasks are called *composite* MarSystems.

A **complete list of all available MarSystems** and detailed documentation of each is provided [here](http://marsyas.info/assets/docs/sourceDoc/html/index.html). That is the comprehensive C++ library documentation, but the information is useful even if you don't intend to use the C++ library directly (but perhaps rather write Marsyas scripts, as introduced below).

## Networks

A composition of MarSystems that exchange data to be processed is called a MarSystem *network*. A network is composed solely by means of adding MarSystems as children to *composite* MarSystems. Inputs and outputs of MarSystems are never explicitly connected, but the way data is passed between MarSystems is determined by the type of their parent. Examples of composite MarSystems: Series, Parallel, Fanout,.. The type-names are quite self-explanatory.

## Processing

An individual MarSystem can be asked to perform processing on a particular piece of data by invoking its *tick* method and providing input and output data locations as arguments. Once a network of MarSystems is composed, the complete network can be processed by invoking the same *tick* method on the single top-most MarSystem of the network hierarchy. The composite MarSystems will take care of invoking processing of their child MarSystems, while passing data produced by some MarSystems to other MarSystems.

Typically, large amount of data is processed by breaking it down into small chunks and *ticking* the network multiple times, each time providing the next chunk as input.

## Controls

Each MarSystem has a set of parameters that affect its processing: they are called *controls*. Each control has 2 attributes: a name and a type. Possible types are:

- `mrs_bool` - true or false
- `mrs_natural` - integer number
- `mrs_real` - real number
- `mrs_realvec` - matrix of real numbers
- `mrs_string` - text

Names and types of controls are defined by the MarSystem the belong to, and can not be changed by the user. The user can set controls to desired values of matching types. Controls can be changed at any time between processing steps (*ticks*) of a network, and will have effect on processed output data the next time processing is invoked (the network is *ticked*).

Moreover, controls of same type (and potentially of different MarSystems) can be *linked* so that whenever one changes, the other one immediately takes on the same value.

# Scripting

The simplest way to use Marsyas is to write Marsyas *scripts*. These are scripts written in the declarative Marsyas scripting language, which  provide very simple ways of specifying compositions of MarSystems. Various Marsyas tools can interpret these scripts, instantiate specified networks, execute the networks, inspect and debug their behavior, etc.

# Inspector

The Marsyas Inspector is a GUI program that interprets a Marsyas script and allows inspection of the declared network by visualizing its structure, allowing tick-by-tick execution, and displaying input, output and control data of any MarSystem in the network at any point of execution.

The inspector is accessible in the following way:

- **Mac OS X:** the Marsyas Inspector app
- **Windows and Linux:** the `marsyas-inspector` executable in the `bin` subfolder of Marsyas

# Command-line tools

The `marsyas-run`tool provides real-time or as-fast-as-possible execution of Marsyas scripts. It interprets a script to construct a network, and then keeps *ticking* the network repeatedly until the `done` control of the top-level MarSystem becomes `true`. MarSystem controls can also be set as command-line arguments to the program. Invoke `marsyas-run -h` to get a description of all command-line arguments.


The `marsyas-run` tool is accessible in the following way:

- **Mac OS X:** the `marsyas-run` executable within the Marsyas Inspector app within the `Contents/MacOS` subfolder
- **Windows and Linux:** the `marsyas-run` executable in the `bin` subfolder of Marsyas
