2026-02-22 00:13

Tags: [[syntax errors]] [[semantic errors]] [[debugging]] [[integrated debugger]] 

There are a couple ways that you can debug. One of them mentioned in this chapter was to use debug statements. It is recommended to use std::cerr instead of std::cout for these because std::cerr is unbuffered while std::cout is buffered. So std::cout doesn't always output to the screen immediately so you might not know if you're looking in the right or wrong place if you're code crashes before the statement has time to output. A disadvantage to these are that they clutter your code and are extra work because you have to remove these statements. However you can use preprocessor conditional compilation to only allow those debug statements to be used when the conditional compilation allows it. These get rid of the extra work of having to remove those debug statements, but they do add more clutter to your code. 

You can also use a logger. A log is a sequential record of events that have happened, usually time stamped. These logs are usually written to a log file and this is great because it's completely separate from the code. 

There is an output stream that C++ provides within the standard library called std::clog that is used to write logging information. However, it writes to the standard error stream (same as std::cerr) and although you can redirect it to file instead, you're better off using a third-party logging tool. One third-party logging tool you can use is the plog logger. 



