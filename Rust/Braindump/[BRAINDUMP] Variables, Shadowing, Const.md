2026-06-20 00:13

Tags: 

Rust Lesson 1 - Variables, Shadowing, Constants (my observations and reasoning towards certain things, could be wrong but it's just what comes to mind):

fn main() {
    let x = 3;
    println!("The value of x is: {}", x);

    x = 4;
    println!("The value of x is: {}", x);

}

This is not allowed because x is an immutable variable. my immediate thoughts here are "does this give us a reason to not need const at all? like in c++ immutable variables are const, constexpr, etc."

fn main() {
    let mut x = 3;
    println!("The value of x is: {}", x);

    x = 4;
    println!("The value of x is: {}", x);

}

This is perfectly fine though. We can have x be a mutable variable and then change it without needing the let keyword. This has me wondering what does let do? does it define a variable? does it allow the type to be inferred like in javascript? i'll find out soon.

fn main() {
    let x = 3;
    println!("The value of x is: {}", x);

    let x = 4;
    println!("The value of x is: {}", x);

}

This is also apparently fine. It is known as variable shadowing. It allows x to still exist but its value is changed to 4. 

fn main() {
    let x = 3;
    println!("The value of x is: {}", x);

    let x = "hello world!";
    println!("The value of x is: {}", x);

}

This is also fine! We can actually change the types of a variable. This is convenient in my opinion. Reminds me a lot of the javascript and python types where I dont have to define a data type, it just infers it for me and is allowed to change it. It also reminds me of the keyword auto in C++ however auto can't change the type later. 

fn main() {
    let mut x = 3;
    println!("The value of x is: {}", x);

    x = "hello world!";
    println!("The value of x is: {}", x);

}

However, this is not okay. Even though the variable is mutable, the compiler expects an integer. I really do wonder why this is the case. I was told that it is because declaring a variable as mutable means that the value can change but not the type. I was also told that you can't change the type unless you are shadowing. 


So it turns out that const does actually exist. It is the same as C++ in the way it is defined, we use the const keyword before tha variable. 

fn main() {
    const MAX_POINTS = 100_000;
    println!("The value of MAX_POINTS is {}", MAX_POINTS);
}

This does not work! But something very interesting is that the compiler tells us what type we might use for this constant. That is amazing, we don't have to guess, the compiler straight up tells us what it could be. I'm really liking how the compiler tells us things, better than any language i've seen so far handle that. 

also one thing I realized was when i did the println! I did 

println!("The value of MAX_POINTS is ", MAX_POINTS);

accidentally and the compiler said the variable isnt used, and then I added the curly braces {} 
and it worked. It got me thinking that the curly braces are definitely important here. It reminded me of string interpolation like in python with fstrings where our variables can be put within curly braces and we can print out the value of the variable within the string. We dont need to put the value in the curly braces though so that raises some concerns like "what if we have multiple variables we want in the output? Does it go by order of variables after the string? Does it go by order of variables declared? or does it not even allow multiple variables? best design choice probably the order of the variables after the comma after the string, that is pretty straightforward but idk yet.

Turns out yeah,

fn main() {  
    let x = 5;  
    let y = 10;  
  
    println!("{} {}", y, x);  
}

the compiler does it by positional arrangement. I do wonder why they chose this rather than Python-style interpolation.

but tangent aside, 

fn main() {
    const MAX_POINTS:i32 = 100_000;
    println!("The value of MAX_POINTS is {}", MAX_POINTS);
}

This is fine after declaring the type. So this makes sense as to why constants are used. This is if we know the specific type already AND we want it to be an immutable value.

also I noticed that we write 100,000 as 100_000, I tried 100,000 and it doesnt work.


The guy asked "Do you think that shadowing is allowed with constants?" 

My guess: No I don't think that shadowing is allowed with constants, because as we saw earlier shadowing allows us to change the value and type of the variable that we had, but since we declared a type already, I don't think the compiler will allow us to change the type and since it is constant we shouldnt be able to change the value.

Actual Answer: 

fn main() {
    const MAX_POINTS:i32 = 100_000;
    println!("The value of MAX_POINTS is {}", MAX_POINTS);

    const MAX_POINTS:i32 = 10_000;
    println!("The value of MAX_POINTS is {}", MAX_POINTS);
}

It does not allow shadowing. It can only be defined once. Unfortunately I didn't get a good explanation as to why. I hope my guess was correct.