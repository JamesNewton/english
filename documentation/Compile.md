From:
https://forums.parallax.com/discussion/163792/plain-english-programming/p4

It takes 17 steps to fully compile a directory (all of the source code for a project must be in a single directory). Here's the top-level code:

To compile a directory:
- Compile the directory (start).
- Compile the directory (read the source files).
- Compile the directory (scan the source files).
- Compile the directory (resolve the types).
- Compile the directory (resolve the globals).
- Compile the directory (compile the headers of the routines).
- Compile the directory (calculate lengths and offsets of types).
- Compile the directory (add the built-in memory routines).
- Compile the directory (index the routines for utility use).
- Compile the directory (compile the bodies of the routines).
- Compile the directory (add and compile the built-in startup routine).
- Compile the directory (offset parameters and variables).
- Compile the directory (address).
- Compile the directory (transmogrify).
- Compile the directory (link).
- Compile the directory (write the exe).
- Compile the directory (stop).

And here's what it does:

1. We start up, which resets some timers so we an see how long each step takes.

2. We read all the source files into memory.

3. We scan the source files looking for "sentence starters" and "sentence enders" and build three linked lists of the TYPE, the GLOBAL, and the ROUTINE statements we find. The ROUTINE records contain routine header information plus a list of the sentences found in each routine's body.

4. We "resolve" the TYPES by recursively examining them all, filling in "parent" TYPES as we bubble back out of the recursion.

5. We whip through the GLOBALs and point them to the appropriate TYPE records.

6. We compile the headers of the ROUTINES, hanging a list of PARAMETERS and a list of MONIKERS (routine-name fragments) on each ROUTINE record.

7. We whip through the types recursively and calculate their lengths -- primitive types first, then user-defined simple types, then user-defined record types including the offsets of fields in those records.

8. We add some "built-in" routines to the ROUTINE list for memory management.

9. We index the ROUTINES for "utility" use. This index is used later as a last resort when a matching routine cannot be found for a call. In essence, it reduces all of the routines to their bare-minimum characteristics.

10. We compile the bodies of the ROUTINES, sentence by sentence, searching for a matching routine header for each. If we find a match, we point that sentence to the ROUTINE record it should call. If we don't find a match -- even in the "utility" index -- then we report the error: "I don't know how to..."

11. We add in and compile the built-in startup routine which calls a specially-named library function to get things rolling.

12. We whip through all the the parameters and the local variables for each routine and fill in their offsets on the stack based on their TYPES.

13. We calculate absolute addresses for all the GLOBALS (including literals that we discovered as we were going along), and for all the ROUTINES.

14. We "transmogrify" everything that needs transmogrification. :) That is, we attach tiny little assembler code fragments to each record that needs them. A little snippet of assembler code, for example, to initialize a global variable, or to push a parameter onto the stack.

```
To transmogrify a fragment (call external):
Attach $FF15 and the fragment's entry's address to the fragment's code.

To transmogrify a fragment (call internal):
Get an address given the fragment's routine.
Attach $E8 and the address to the fragment.
```

15. We "link" the whole shebang together into a PE-style executable in a big memory buffer. That is, we copy all those little fragments of assembler into the proper spots in the file buffer. We ignore anything that isn't used to keep the executable as small as possible. So everything in the Noodle (our standard library) gets compiled every time, but only the parts that are actually used are included in the executable.

16. We write the executable buffer to disk with an appropriate name derived from the directory name where the source files are stored.

17. We stop the last of the timers.

If the programmer selects the "List" command from the menu (instead of "Compile" or "Run") we produce a detailed listing of everything the compiler was thinking about (and generating) as it went along. You can see a piece of such a listing on page 88 of the manual, including a bunch of those assembler code fragments (in big-endian hex; or is it little-endian? I can never remember).

 There are a number of "decider" routines that tell us whether a particular string is an article, or a preposition, or a conjunction, for example. Routines like this:

To decide if a string is any conjunction:
- If the string is "and", say yes.
- If the string is "both", say yes.
- If the string is "but", say yes.
- If the string is "either", say yes.
- If the string is "neither", say yes.
- If the string is "nor", say yes.
- If the string is "or", say yes.
- Say no.

But there are only a handful of those. The bulk of a program's vocabulary (and grammar, for that matter) is inherent in the TYPES, GLOBALS, and ROUTINE HEADERS that the programmer has coded. And those are indexed using hash tables (with linked lists for overflow) so we can the find the things we need quickly.
