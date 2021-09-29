# Unreal Engine Blueprints Style Guide

Unreal Engine Blueprints (visual programming tool) style guide.

## Naming

Use [PascalCase](https://techterms.com/definition/pascalcase) for naming anything inside blueprints, including variables, functions, delegates and such.

Names MUST ALWAYS be something meaningful, even if you have to name it something long in order to keep it understandable. It's ok to name stuff like "CurrentAngleBetweenCameraAndChar".
Do not use meaningless names or too short names, like "Temp", "Var" or "X".

Namings should be in English.

### Variable naming

Variable name should describe a value, that is stored inside.

Examples:

```plain
Kills
ObjectLocation
AbleToUseObject
```

### Function naming

Function namings should describe an action, that a function do.

Always start a function name with a verb.

Examples:

```plain
FindItemInInventory
GetSpeed
CalculateMovementVector
HideOptionScreenOnInteractionDestroy
```

### Dispatcher naming

Use "On" prefix when name a dispatcher.

Examples:

```plain
OnDeath
OnItemAdded
OnGameEnded
```

## Macros

Do not use macros. Almost everything you can do by using functions.
The only reason you need a macro, is to use some kind of reusable and collapsed flow control nodes.

## Functions and Events

Almost everything should be under functions. This rule created just to organize your code better. If you use functions for everything, you will have all of your code separated by meaningful actions and you will be able to easily find a piece of logic inside "Blueprints" panel in a form of a convenient list.
If you will make everything inside an EventGraph, than you get a huge chaos, where everything is hard to find..

Functions are also have next advantages over Events:

- Local variables (Temporary variables, which are available only in a scope of a function)
- Function parameters exposed as local variable nodes (Search for Get*ParamName* in context menu)
- Functions can have return values
- Function flow can be explicitly interrupted using Return Node
- Function can be pure (allow to call without execute pins)
- Functions can be private (you are not able to call this functions from other blueprints)
- Functions can be const (prevent a developer from modifying the state of a Blueprint)

You should also convert default events to functions, as well as overwritten functions without return values. That way you see these events in function list and have access to local variables.

![Blueprints event convert to function](/img/Functions_EventToFunc.png)

You still can use events when it is not possible to use functions (Exposed dispatcher events, RPC functions), or when you need to explicitly show the flow of logic with dispatchers.

## Delegate binding

You can bind a function to a delegate as well, and it is preferred. Use CreateEvent node to bind a function.

![Blueprints function dispatcher binding](/img/Functions_Binding.png)

## Return nodes

Use return node to exit a function when it is needed. Do not create variables, that determine function exit later. It is more often used with ForLoop, when you need to return a value when it is found, instead of using ForLoopWithBreak.

Wrong way:
![Blueprints function return Wrong](/img/Functions_LoopReturn_Wrong.png)
Ok:
![Blueprints function return Ok](/img/Functions_LoopReturn_OK.png)
This code is cleaner and you do not use extra variables.

## Parameters and variables

Use function parameters as a Get node. Do not pull wires over a function graph.

You can use wired connection though, when target is near a function start.

![Blueprints parameters Wrong](/img/Functions_Params_Wrong.png)
![Blueprints parameters Ok](/img/Functions_Params_OK.png)

Use "L\_" for a local variables, so you always know, if it is a function or a blueprint variable.

## Pure vs Impure

In blueprints, we have two kind of functions: Pure and Impure. By default, function is Impure, which means, that it has executable pins and should be inserted in your logic flow. Pure, on the other hand, does not have any executable pins and should be used to calculate Impure function parameters.

![Blueprint Pure and Impure functions](/img/Functions_PureImpure.png)

There is a simple rule, when you must use pure functions. If your function does not modify any state of any objects, and used to calculate or return some values, than it should be a Pure function.
Very often you can determine a Pure function by it's name. If it starts from "Get", "Find", "Calculate", "Is" or something like this, than it must be a Pure function.

Note, that Pure functions get called every Impure function call. It means, that one Pure function note can be called as many times, as many Impure functions it is connected to. Be careful!

## Flow Control

### Branch

You can use False only Branch, even though it is not a good practice in text-based code. In Blueprints, it is often more readable to use False execution only, than placing "Not" node before a Branch condition. Using branch with "Not" is also a good option.

![Blueprints Branch](/img/Flow_Branch.png)

### Flow Control nodes

As all of the code should be under functions, you are not able to use some Flow Control nodes, that remember their state (FlipFlop, DoN, Gate e.t.c) or are asynchronous (Delay, RetDelay, e.t.c).

There is also no need to use Delay. Prefer Timers, that call other functions over time. This also has nest benefits:

- You can stop Timer before it finished. Use TimerHandle -> ClearAndInvalidateTimer
- You can loop timer, which allows you to call a function repeatedly.
- Timers work in functions, as well as in EventGraph.
- You can dynamically change end function call in runtime (When using SetTimerByFunctionName)

You can use Delay in some visual-related Blueprints, when it just more convenient to use delays.
You can also use Delay(0) to skip one frame, when it is needed. This is useful, for example, when you need to wait Actor components to initialize from other component, which job depends on owning Actor initialization process (BeginPlay in a component may execute earlier, than BeginPlay in own Actor or other components).

### Validation

Validation is used to check, whenever variable points to an existing object.

You should use "Validated Get" node when you need to prevent flow execution when pointer is not valid.
You can turn a simple "Get" node into a "Validated" one by RMB > Convert To Validated Get;

When validating after Set or output node parameter, use IsValid node.

![Blueprints validation](/img/Flow_Validation.png)

## Alignment and Look

The main rule here, is to make as less wire crossing as possible by caching values or using reroute nodes.

If you need a function result value in multiple places, cache the return value and use it by Get node. Same when you need a result value after couple of nodes after.
Do not pull wires over whole graph!

![Blueprints cache result](/img/Functions_Cache.png)

Align nodes with each other, so your code would look much cleaner and easier to read.

![Blueprints nodes align](/img/Look_Align.png)

To align, select nodes, which you want to align between each other and press a hotkey (Hotkeys can be found by RMB on a node > Node Align).

## Comments

You should comment a code, which behavior or existence reason is not obvious. Comments should answer "Why" question. Do not spend time on comments with a description of what your code does, as it should be clear by simply reading your code.

You can also write some thoughts, that can help other developers to understand, why you did something there.

Use your team's primary language in comments, as long as it uses Latin alphabet. Otherwise prefer English comments.

Note, that you can comment some area of you code, as well as some specific node.

You can use comment prefixes, so you could easily find some problems inside you project (by using global search).

BUG - There is a known bug in commented area.
TODO - There is something to fix, add or refactor in commented area.

Comment colors are not necessary and use them on demand of your management.

## Other

### Timers

Prefer "SetTimerByEvent" over "SetTimerByFunctionName". "SetTimerByEvent" will check function binding on compile time, so you do not misspell function name.
The only time you should use "SetTimerByFunctionName", is when you need dynamic function binding in runtime.

![Blueprints timer by event](/img/Other_Timers.png)
