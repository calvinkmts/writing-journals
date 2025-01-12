# The Many Ways of Creating Class in Python 

## Background

So, in my current position, all of our code is written in Python.

Now python itself can be written for a procedural code like (AWS Lambdas) and OOP (Object-Oriented Programing)

Now as for lambdas itself, you can get away with not having class or some sort of contract because it is supposed to be simple (I believe with the limitation of the runtime itself to a 15 mins maximum, and with AWS Glue having to have multiple file or modules is very tedious process. 

Other than that we also have projects that is seemingly medium to large scales, while depending of the context of the project it's not always a necessary to have class, however it will helped with type hinting or static type checker, which I believe if the code itself is obvious and not many abstraction layer we can get away with not writing any class. 

## What I Really Like When Writing Classes

I always go for a composition over inheritance, because it's more easier to manage the newly created class.

This came with experience, when I was learning to use Unity the game engine, it was already a massive framework / game engine, so the tutorial kinda give me the code, okay write this new PlayerClass extends UnityClass, which I don't get it what does it do? what property or methods it has in it, which If you are not very proficient it became quite tedious having to look the documentation or waiting the build saying that you are overwriting some function from the base class which later breaking things.

And it happened recently, when in the project suddenly someone inherited a BaseModelClass from Pydantic to our dataclass models, which we found out the hard way that it broke some of the code. 

This will be the theme of this article, outlining which library will I use to write a class. 

## The Goals of Writing a Class

So before going any deeper, why write class itself in the first place. 

My decision when writing class, is if I needed to save some state. If I have many variables that I need to group to make it easier to read the context of the code. 

So yes, I advocate not to jump writing a wrapper class to use another library which I found quite a lot in my time while writing python recently. 

-- explain the code that bothers me... 

