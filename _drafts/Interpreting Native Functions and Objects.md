# Implementing Built-in Objects and Functions in an Interpreter

Someone on [r/Compilers](https://www.reddit.com/r/Compilers/comments/bo4bye/extend_the_language_in_crafting_interpreters_to/) asked a great question about implementing interpreters. It is a strange fact that despite the overabundance of books on compiler construction, almost no literature directed at a general programmer audience discusses the implementation of interpreters. In the small handful of texts that include anything about interpreters, the discussion is almost always limited to tree-walking interpreters, despite every widely used interpreted language in existence being implemented using a bytecode interpreter.

Enter [Crafting Interpreters](https://www.CraftingInterpreters.com), the prodigious brainchild of the brilliant Bob Nystrom, and the only book I know of that is about interpreters specifically, culminating in an honest-to-goodness bytecode interpreter. It's written in a very accessible way and could easily serve as a first book on compilers/interpreters. But in a way, that's also its weakness. Almost every compiler book is meant to be a first book on compilers, but no major compiler is implemented using just the elementary content an introductory text can cover. 

This article is about a topic that would likely appear in a *second* compiler book, not an introductory text:

> I'm wondering how to extend [my bytecode interpreter] so it has a builtin list, map, etc.

A comprehensive answer needs to discuss two distinct needs: built-in objects and native function calls. What follows is my attempt at a succinct but useful answer. I point toward the implementation in Crafting Interpreters, and I walk through the implementation in Bob Nystrom’s [Magpie](https://github.com/munificent/magpie).

## Crafting Interpreters

[Crafting Interpreters](https://www.CraftingInterpreters.com) covers both topics, but it's easy to miss. Here's where to look.

### Objects

For objects, you will want to study [Chapter 12]([http://craftinginterpreters.com/classes.html](http://craftinginterpreters.com/classes.html)), especially [12.2](http://craftinginterpreters.com/classes.html#class-declarations) and 12.3. You can just define subclasses of `LoxClass` and `LoxInstance`. For the AST tree-walking interpreter, you might want to create `Stmt.Class` nodes, or an appropriate subclass or sister class of `Stmt.Class`.

### Native function calls

Check out [10.2 Native Functions](http://craftinginterpreters.com/functions.html#native-functions) section of Crafting Interpreters, which walks you through exactly how to do this. The upshot is:

> If we wanted to add other native functions—reading input from the user, working with files, etc.—we could add them each as their own anonymous class that implements LoxCallable. 

## Example Implementation

The author of www.CraftingInterpreters.com, Bob Nystrom ([u/munificent](https://www.reddit.com/user/munificent/submitted/)), has a few other great language interpreter projects to learn from. [Magpie](https://github.com/munificent/magpie) is written in C++ and is the closest thing to the Java interpreter of Crafting Interpreters. Wren is implemented in C and has a more sophisticated system for handling native function calls. As a consequence, it is also much harder to understand, but if you’d like to try, [maybe start here](https://github.com/wren-lang/wren/blob/master/src/include/wren.h#L296). I walk through Magpie's implementation of native functions below.

### Magpie

Here’s how u/munificent’s Magpie interpreter does it (slightly edited for length and readability). 

First we define an enum for “native” things:

```c++
// Identifies classes that are defined in core lib but have a native C++
// object class. The VM stores a reference to each class object so that it
// can get the class for a given C++ object.
enum CoreClass{
  // core
  CLASS_BOOL = 0,
  CLASS_CHANNEL,
  CLASS_CHAR,
  CLASS_CLASS,
  CLASS_DONE,
  CLASS_FLOAT,
  CLASS_FUNCTION,
  CLASS_INT,
  CLASS_LIST,
  ⋮
  etc.
};
```

The interpreter stores and retrieves the names and function pointers in vectors. This is a relatively primitive data structure to use for this task. Usually a variation of a hash map is used for this. But whatever container is used, all we really need is a way to initially define a native function and a method of looking up the native function by (string) name:

```c++
// The true signature of a native function. Good call on the typedef.
typedef gc<Object> (*Native)(VM& vm, Fiber& fiber, ArrayView<gc<Object> >& args, NativeResult& result);
  ⋮
// Registers the name and (pointer to the) function.
void VM::defineNative(const char* name, Native native);
// Return index into function pointer vector if found.
int VM::findNative(gc<String> name);
// Fetches function pointer at index.
Native getNative(int index) const;
```

A native function looks like

```c++
// The signature of name##Native.
#define NATIVE(name) gc<Object> name##Native(VM& vm, Fiber& fiber, ArrayView<gc<Object> >& args, NativeResult& result)
  ⋮
NATIVE(intPlusInt){
  return new IntObject(asInt(args[0]) + asInt(args[1]));
}
```

where `NATIVE` is a macro expanding to the correct but long function signature. They are registered at startup:

```c++
// Registers name and pointer to name function as a native class.
#define DEF_NATIVE(name) vm.defineNative(#name, name##Native);
  ⋮
DEF_NATIVE(intPlusInt);
```

Native functions need to hook into the machinery of the interpreter, which is tracked with an enum:

```c++
// When a native function returns, this describes how the return value should
// be used.
enum NativeResult{
  // A normal return value. It will be the result of the native expression.
  NATIVE_RESULT_RETURN,
  // An error that should be thrown.
  NATIVE_RESULT_THROW,
  // The native has pushed a call frame on the stack, so it doesn't have a
  // return value (yet).
  NATIVE_RESULT_CALL,
  // The native is suspending this fiber. It will be resumed later.
  NATIVE_RESULT_SUSPEND
};
```

Then, in the opcode switch of the interpreter main loop, a dedicated opcode handles native calls:

```c++
case OP_NATIVE: {
  Native native = vm_.getNative(GET_A(ins));
  ArrayView<gc<Object> > args(stack_, frame.stackStart);
  NativeResult result = NATIVE_RESULT_RETURN;
  gc<Object> value = native(vm_, *this, args, result);

  switch(result){
    case NATIVE_RESULT_RETURN:
      store(frame, GET_C(ins), value);
      break;
    case NATIVE_RESULT_THROW:
      // TODO(bob): Implement this so natives can throw.
      ASSERT(false, "Not impl.");
      break;
    case NATIVE_RESULT_CALL: {
        gc<FunctionObject> function = asFunction(args[0]);
        // Call the function, passing in this method's arguments except
        // the first one, which is the function itself.
        call(function, frame.stackStart + 1);
        break;
      }
    case NATIVE_RESULT_SUSPEND:
      return FIBER_SUSPEND;
  }
  break;
}
```

And there you have it! 

## Conclusion

Magpie's implementation really is how many major interpreted languages solve the problem of built-ins. But it still has a few limitations:

* Native functions cannot call Magpie functions. 
* Native functions must be available at compile time. That's the compile time of the interpreter, not of the interpreted code!

Wren's FFI solves the second problem and has a way of getting around the first with closures. It is a good exercise to see if you can come up with a design for Magpie that overcomes these limitations. 