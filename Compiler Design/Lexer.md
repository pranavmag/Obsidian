2026-04-16 02:40

Tags: 

### Tokens

Lexical analysis is the first phase of a compiler. This is where raw source code is read one character at a time and grouped into tokens. Tokens are the words and punctuation of the programming language.

For example, 
int x = 3; There are five tokens in this example.

int - keyword
x - an identifier
'=' - an assignment operator (obsidian doesn't allow me to equals sign without side effects so i just added single quotes around it)
3 - an integer literal
; - a statement terminator

You would need to have your token types and your specific token.
Each token bundles together four things: its type, the raw text (lexeme) it was matched from, an optional literal value, and the line number where it appeared. Each of the individual words are lexemes but they become tokens when you bundle it together with the other data.

We can store all these keywords and types in an enumeration to be able to more easily access them later for different tasks.

We need to be able to track errors by telling the users where it occurred. We need to be able to create tokens that can easily allow that kind of tracking. We need to wrap this data in a struct or class to identify what these tokens have. 

As we said earlier:
Token Type
The Lexeme
A Literal Value
Line Number


### Scanning

Now we essentially need a loop here to walk through the source string character by character and group those characters into tokens. The loop continues with every single character that we have in our source code. 

This is similar to regular expressions. Let's take a look at this regex:

![[Pasted image 20260512192703.png|249]]

This regex means that for the first letter you can use any value from a-z, A-Z, or an underscore. Any following letters can use the same but with the inclusion of 0-9. This matches most languages that don't allow a number to be used as the first character of the identifier.

These rules that determine how a language groups characters into lexemes are called its lexical grammar.

So for example if you have something like print 123;
It will be split up into 

PRINT
NUMBER(123)
SEMICOLON
EOF

### Entering Program

We need to set up our main to actually run the logic we write for the lexer/parser/compiler logic. Since we're reading individual characters of a source we need a function that takes in a string that carries that source. 

What we can handle right now that is most important is command line handling, file reading, REPL handling, and error codes.

We need to know when errors occur, so having a shared state that can flag that error as true is necessary. 


We also need to have a way to handle files, so we need to know the file path that the user is passing in (very similar to how we did it in the emulator to take read .bin files). We can use std::ifstream to read that file.

You have to read the whole file as well using std::ostringstream. std::ostringstream is like a string builder that stores output in a string. You need to copy the entire contents of the file into the buffer and then return that built string. You must also check for errors.


We also need to handle the REPL to run prompts where the user doesn't need to use a file. The user can enter the REPL and type > print 3;
You need to set an infinite loop for this and allow one full line from keyboard input to be read.
You also have to handle EOF to see when input ends. For Windows it would be Ctrl+Z and Ctrl+D on Linux and MacOS. Then you can run the line typed by the user. Then reset errors.


Now for int main. You start off by passing in int argc and char* argv[]. argc is the argument count and argv is the array of arguments. argc should take in two arguments, for example:

C test.C 
argc = 2

argv[0] = "C"
argv[1] = "test.C"

Any argument count past 2 is too much. If the argument count is 2 then run the file with the second element of the argv array which will be the file. If the argument count is less than two you can just run the prompt.



### Error Reporting

You want to report for errors like the line number where the error happened, where the error is, and the message for the error. For example:

[line 4] Error at '*': Unexpected token.







