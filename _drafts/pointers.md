---
layout:     post
title:      "A Crash Course in Pointers"
subtitle:   "A (hopefully) simple crash course on pointers."
date:       2015-06-30 10:00:00
author:     "Ganesh Datta"
comments:   true
---

Pointers - *, **, &, and more - so many symbols meaning so many things. What these all have in common is that they are operators used when dealing with pointers in C and C++. But what are pointers?

Before understanding pointers, we need to understand variables and their connection to memory. In your computer, memory is where variables and their values are stored. Some types of variables are bigger than others - for example, a `char` is 1 byte, whereas an `int` is 4. A byte is 8 binary bits, and is how memory is measured at a basic level. For example, an integer with value 129 is represented in binary as 0000 0000 0000 0000 0000 0000 1000 0001. This integer needs 8 bytes to store the entire value. When you declare an `int`, like `int somenum = 129`, you are *setting aside a piece of your memory for this variable to store the value*. Keep this in mind, it’ll be important later!

Now, that’s all great, but what does that have to do with pointers? Let’s start with examining an issue we face in C and C++, by looking at the following function.

{% highlight c %}
void aMustBeFive(int a){
	a = 5;
}
int main(){
	int myMainVar = 7;
	aMustBeFive(myMainVar);
}
{% endhighlight %}

What’s going on here? As you see, the goal of `aMustBeFive` is the change the value of `a`, which is passed in as a parameter to the function, to 5. What’s the problem? When you pass in an int, for example, you are *passing it by value*. The function `aMustBeFive` gets the value of whatever `a` is set to when you call it, and it creates it’s own version of the variable to use within the function. So, when you update `a`, you’re not changing the original variable you passed in, but a copy of the variable local to the function! In this case, nothing ever happens to `myMainVar`!

How do we solve this problem? As I mentioned earlier, when you declare a variable, you’re setting aside a piece of memory for it, where the value is stored. What if when we want to change the value of `a`, we go find where it’s located in memory, and change the value right there - directly in memory? Now that’s a novel idea… and that’s exactly what pointers are!

Everything that’s stored in your memory has what’s called an address. It’s exactly what it sounds like. I’ll try to explain this with an example. Pretend I have a special box at 123 Pointer Ln. Inside this box is my ID number. How do I change my ID number? I need to go to 123 Pointer lane, open up the box, and change the number stored inside the box. This is exactly how pointers work.

In memory, addresses look a little weirder. They’re generally along the lines of 0x12345678. This is a number that is 8 bytes long, and *points* to somewhere in memory. By accessing this location in memory, I can change what’s stored there! With this in mind, let’s write a new function that actually works. Don’t worry if the syntax doesn’t make sense, I’ll explain it shortly.

{% highlight c %}
void aMustBeFive(int * a){
	*a = 5;
}

int main(){
	int myMainVar = 7;
	aMustBeFive(&myMainVar);
}
{% endhighlight %}

*”Slow the fuck down!”* ~ You.
Alright, let’s take this one step at a time. First off, you can see that I’m no longer passing `myMainVar` to `aMustBeFive` - I’m passing `&myMainVar`. What does the `&` do? It gets the *address* of the variable (where it’s located in memory), in this case the address of `myMainVar`. It’s like to asking me “Ganesh, where’s your special box?” - I reply “123 Pointer Ln”. Similarly, `&myMainVar` will return something along the lines of `0x12345678`, which is the address it’s located in memory. To change it’s value from 7, we need to somehow go to that address in memory and change it’s value.

Now, you see that `aMustBeFive` takes in an `int * a`… wtf is that?? It’s a pointer to an integer - the name of the variable is irrelevant. It’s basically a business card of the variable being passed in to the function with its address on it! A pointer is nothing but a special type of variable that *points* to a location in memory. If I say `int * a`, it’s a way of saying that if I drive through memory until I find the address that `a` points to, I’m going to find some integer!

But we’re passing in some funky `&` shit to `aMustBeFive` - what does that mean? When we take the address of a variable, we are implicitly creating a pointer - a pointer points to some location in memory, so if you ask for a variable’s address, it’ll give you a special variable that will allow you to go to the address directly! If you asked me, “Ganesh, where’s your special box”, I’d be giving you a teleportation device that takes you directly to 123 Pointer Ln. 

Alright, so now `a` is a pointer, not an `int`! That means the value of `a` is something like `0x12345678`. What happened to my 7??? Remember, `a` simply *points to a location in memory*. I have to now find that location in memory and open the special box to see what’s stored there. To do that, we do something called *dereferencing*. By dereferencing a pointer, you get the value stored at that location in memory. So, if you were to dereference `a`, you’d be saying - “Hey computer, go open up the box located at 0x12345678 so that I can see what’s there.” The syntax to do this is `*a`. 

Now, the function starts to make more sense! `aMustBeFive` receives a pointer to some integer - it doesn’t care what it is, it just knows that if it goes to the location that `a` points to, it’ll find some integer. Then, it dereferences the pointer by doing `*a`. This opens up the box at the address that `a` points to, allowing it to change whatever is stored there. It finally shoves the value 5 into that open box! 

Does this change the value of `myMainVar` though? Absolutely! In `main`, you’re passing the address of `myMainVar` to `aMustBeFive`! This is saying, “aMustBeFive, here’s the address of my secret box - I’m entrusting you with a pointer to that location. You can now change whatever is there directly!” 

By passing a pointer, we solve the problem of passing by value - instead simply creating a new variable with the same value, we get a special tool that allows to go to wherever the original variable is found in memory and change it’s value directly!

Phew! Pointers are super complex and they take a while to get used to. Once you get the the hang of them though, they’re so powerful and you’ll walk out feeling like a fucking master. Code on!