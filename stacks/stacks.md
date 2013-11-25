The “stacks” esoteric programming language (name may change) transports the UNIX philosophy of “do one thing and do that well” to programs:
“Each function should contain exactly one instruction.”

As it turns out, that doesn’t work (because running the next line requires running one instruction), so this language is so generous and provides you with *two* instructions.

Here’s a simple hello world:

    main
        STDOUT < "Hello, world!\n"
        PC < NULL

`PC` is the *Program Counter*, and setting its topmost value to `null` (the `NULL` stack returns infinite `null`s) will stop execution.

And here’s an `echo` (actually, in shell terms, a `cat` without arguments):

    main
        STDOUT < STDIN
        // PC < main // unnecessary

It reads a line from standard input, writes it to standard output, and then repeats.

*stacks* is, as the name suggests, a stack-based programming language: All data is handled using stacks.
Stacks’ names are UPPERCASE, while function names are lowercase.

The syntax for pushing and popping is very intuitive:

    source > TARGET
    TARGET < source

Here, `TARGET` is a stack, while `source` can be a stack, a literal, or a function.

You can also peek:

    SOURCE <=> > TARGET
    TARGET < <=> SOURCE

This puts the topmost value from `SOURCE` onto `TARGET` without removing it from `SOURCE`. (In case you can’t tell, the `<=>` represents an eye – the eyelids aren’t quite closed, so you can peek through them.)

Here’s how conditionals work:

    CONDITION > :
      <then>
      <else>

The topmost value is popped off `CONDITION` (which can be any stack). If it’s nonnull and nonzero, the `<then>` line will be executed; otherwise, the `<else>` line will be executed.

Of course, you’ll need something to put on the `CONDITION` stack. I think it’s time to introduce you to *stacks*’ *special stacks*:

* `PC`: the program counter. Each time a function ends, *stacks* looks at the topmost value of `PC` (without removing it!) and executes that function; if it’s not a function, the program terminates.
* `STDOUT`: Pushing a string, number or function onto this stack will print that string, number or function’s name to standard output.
* `STDERR`: same as `STDOUT`, except it prints to standard error.
* `STDIN`: popping or peeking from this stack reads a string, number, or function from standard input, depending on the type of what was last pushed onto it.
* unary operation stacks: These stacks all “take one argument” to return a “boolean” (a number), meaning that if you push a value on them, they will replace it with the result of the respective operation, which you can subsequently pop or peek.
  * `NOT`: inverts a “boolean” variable.
  * `ISFUNCTION`: determines if the argument is a function.
  * `ISSTRING`: determines if the argument is a string.
  * `ISNUMBER`: determines if the argument is a number.
  * `ISNULL`: determines if the argument is `null`.
* binary operation stacks: Similarly to the unary operation stacks, these stacks all “take two arguments” to return a “boolean”, meaning that if you push two values on them and then read (pop or peek) from them, they will return the result of the respective operation on the two operands (where the first pushed value is considered the “left” operand). If you just push one value onto them, you get a sort of “curried operation”.
  * `EQ`: tests if the two operands are equal.
  * `LT`: tests if the left operand is less than the right operand.
  * `GL`: tests if the left operand is greater than the right operand.
  * `LTE`: tests if the left operand is less than or equal to the right operand.
  * `GTE`: tests if the left operand is greater than or equal to the right operand.
  * `ADD`: adds the two operands.
  * `SUBTRACT`: subtracts the right from the left operand.
  * `MULTIPLY`: multiplies the two operands.
  * `DIVIDE`: divides the left by the right operand.
  * `MODULO`: calculates the remainder of dividing the left by the right operand.
* `NULL`: this stack is always empty. (Empty stacks, when read, return `null`.)

Let’s see another example: We read a number from standard input and print it according to the “Fizz Buzz” rules.

    main0
        STDIN < 1 // tell STDIN that we’re expecting a number
        PC < main1
    main1
        INPUT < STDIN // the stack INPUT gets created on-the-fly
        PC < main2
    main2
        MODULO < INPUT
        PC < main3
    main3
        MODULO < 3
        MODULO :
            PC < nofizz0 // remainder is nonzero, not evenly divisible
            PC < fizz0 // remainder is zero, evenly divisible
    nofizz0
        MODULO < INPUT
        PC < nofizz1
    nofizz1
        MODULO < 5
        MODULO :
            PC < nofizznobuzz // remainder is nonzero, not evenly divisible
            PC < nofizzbuzz // remainder is zero, evenly divisible
    fizz0:
        MODULO < INPUT
        PC < fizz1
    fizz1:
        MODULO < 5
        MODULO :
            PC < fizznobuzz // remainder is nonzero, not evenly divisible
            PC < fizzbuzz // remainder is zero, evenly divisible
    nofizznobuzz
        STDOUT < INPUT
        PC < newline
    nofizzbuzz
        STDOUT < "buzz"
        PC < newline
    fizznobuzz
        STDOUT < "fizz"
        PC < newline
    fizzbuzz
        STDOUT < "fizz buzz"
        PC < newline
    newline
        STDOUT < "\n"
        PC < NULL
