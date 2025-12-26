# Solid Principles of Object-Oriented Design

## A Lecture by Robert Martin (Uncle Bob)

Good evening. My name is Kyle Jensen. I'm on the faculty here at the Yale School of Management, and I'm also the director of entrepreneurial programs here. Our guest tonight is Robert Martin, who's going to talk about SOLID principles of object-oriented and agile design. We are very lucky to have him here.

He's here at a great time for Yale. As you may know, the university is investing a great deal in entrepreneurship. This is creating, or mixing with, the university's traditional expertise in computer science and the theory side, producing all sorts of great things on campus.

For example, two years ago, Yale started the tech bootcamp. Hack Yale started a year ago. Yale Hack brought together a thousand developers from around the country for a two-day hackathon. This semester, Daniel Boddy changed the introduction to programming course here, which is the core programming course for non-majors, to be mobile application development.

Daniel and I are teaching together next year in the spring. Computer Science 113, which is entrepreneurship and programming. We have a course on management of software development here at the Yale School of Management.

This is to say that there's a great interest at Yale in software development, and it transcends a couple different disciplines. It's here at the School of Management and the Department of Computer Science. It's in the School of Engineering and all sorts of places. Across different clubs, organizations like the Yale Entrepreneurial Institute, Hack Yale, Entrepreneurship Club here at ISTY Hack, all these kinds of organizations.

It's not just Yale. There are a couple logos up there that are not at Yale but are indicative of the activity in this space in the local community. We have to thank Continuity Control for the food tonight. It's a local startup and Ruby shop filled with wonderful hackers and developers who are very active in New Haven IO, which is a local developer group that I've been honored to be a part of.

I worked with Dan and Joel and many other people who are here to organize a variety of events for New Haven IO. I'm very happy to say that the plurality of people here tonight are actually probably not from Yale, which I think is really important. Traditionally, Yale has this kind of town and gown problem, where there's not much interaction. I think that's a real problem.

This is an area in which everybody has a mutual interest and excitement. It's wonderful to have events like this. We're very happy to have Bob here to speak to us. I'm gonna invite Dan to come up and introduce himself. Dan, I will introduce you. Dan is a developer at Continuity Control. He has been an organizer of the local Ruby group for many years and organizer at New Haven IO. I think he's on the board of New Haven IO. He is essentially a very active local developer, very supportive of the local community. I think that's wonderful. That's how our friendship has grown in that context. So Dan, thank you.

All right. I should just be projecting. Sorry. All right, Dan Bernie, Continuity, New Haven IO, summary version. All right. So like I did say, we are hiring, and we all have little Continuity lapel pins. We do some pretty interesting things. We aim to have really clean code. We have some really interesting projects coming up with machine learning, interesting stuff like that. So we're hiring. Come talk to us.

## Introduction to Software Engineering Challenges

Software is making software. It's a weird business. It's a messy business. Those of you who are students may not have found this out yet because when it's just for fun, it's a lot of fun. But when it matters, when you have people paying you for the software that you're writing, or people using your software, or God forbid, people whose lives depend on your software, it's a weird business. Sometimes you can feel like we're a bunch of kids running around with our shoes untied, just hoping you don't fall down.

Luckily, we have an uncle looking out for us, giving us a kick in the butt every once in a while when we need it. Uncle Bob has been doing quite a number of things over his career. He helped write and sign the Agile Manifesto, which really helped the industry rethink the way it looks at working with business people and the way it plans software. How much we can know about a piece of software before we build it, and how wise it is to bet a lot on that.

He's helped us look at the way we organize our code, from down to how do you name that variable and that method, all the way up to how do you organize pieces of applications and make them talk to each other. We're going to talk tonight, or he's going to talk tonight, about a spot right in the middle about how you organize your objects and have them relate to each other.

He's helped us understand new programming languages and why they matter and why they help. He's been writing books about all these things along the way and speaking at events like this all around. We owe a lot to him. He's raised the overall level of our field quite a lot. He's kind of journaled it all along the way. So, hoping I haven't built him up too much, but with that, Bob...

All right. Let's see. Do I need to turn the switch on? Or do the audio people have this going? Switch on? Yeah. All right. Oh, I talk quietly, there. How's that? Oh, good. Okay.

## Water: An Analogy for Understanding Dependencies

How many of you are programmers? Look at that. Yeah, programmers. Is there anyone who is not? You're not? You're what? Online product? That's not a programmer. No, not. Okay. All right. Most of you are.

What is water? A naturally occurring substance. Good. Very general definition. Soft or hard? Water itself is soft. It's the stuff in it that makes it hard. What's the chemical formula? H₂O. Which means two hydrogen, one oxygen. What does the molecule look like? Mickey Mouse. Very good.

So a water molecule has a big oxygen atom and two hydrogen atoms, just like Mickey Mouse. The angle here, if I remember correctly, is 103 degrees for quantum mechanical reasons having to do with God knows what. Why do these three atoms stick together?

Now, before you give me the covalent bond answer, think about what those atoms are. That's a proton for the hydrogen, with a positive charge, and another proton for the other hydrogen with a positive charge. How many protons in the oxygen atom? Eight. Eight protons in the center of the oxygen nucleus. Then around those protons is this cloud of electrons: one electron for the hydrogen, one electron for the other hydrogen, and eight for the oxygen.

That cloud of electrons has a negative charge. So these three atoms should repel each other because they're surrounded by negative charge. Why do they stick together? What's holding them? They share. It's classic. Really sure. What do they share?

Let's look carefully now. First of all, if the temperature is low enough, if these atoms aren't moving fast enough, they do repel each other. They're not going to get near each other. You have to get them moving fast. You have to get them to bump into each other. If you get them to bump into each other, something really interesting happens, which is this:

First of all, you have positive charge here, positive charge here, and positive charge here. Where do the electrons want to go? They want to be as close to all those positive charges as they can get, which happens to be right here. So you get a preponderance of negative charge between the protons, and the negative charge holds the protons together. You just have to get them close enough, and then they kind of go slip, and they snap together.

Now, you mentioned that they share. Well, they do share. Why? Because there's some bizarre quantum mechanical effect that has to do with the Pauli Exclusion Principle, which says that the outer shell of the oxygen atom can have as many as eight electrons in it. It's only got six, and the two electrons from the hydrogens will just snap into place there. That's not the reason they stick together. They stick together because of this charge. They will fit because of the quantum mechanical effect. That is a covalent bond.

A covalent bond happens when you manage to get the bulk of the negative charge to hold the positive charges together. Of course, the electrons flit around here, and it's a big probabilistic thing. They're really all over the place, but they will spend more time there, and they hold the whole thing together.

Now look at this. I bisect the molecule, cutting Mickey Mouse's head in half. Where's most of the negative charge? It's above the line. The molecule does not have its charge evenly distributed. It is more negative above the line. It is more positive below the line. The molecule has a dipole, a charge across it.

My wife gets mad at me when I put the 'T' on the end of "across." What that means is that the molecule wants to rotate and stick to anything that has a charge, no matter what the charge is.

Why does water stick to your hand? Why does water get wet? Because all the molecules rotate and find some little charge on your hand, and they stick to it electrostatically.

Why does water dissolve things? Because anything that has a little charge, the water molecule will stick to.

Why is water good for washing? Because it can stick to dirt.

Who's done the cute little trick where you take a faucet and you take a really thin stream of water? You have to get this as thin as you can get it, but without drips. It's got to be a nice thin stream of water. Then you get a balloon and you rub the balloon on your head. You take the balloon and you put it there, and the water just goes "whee!" Attracts. The balloon attracts a lot. Who's done that one? Dazzle your children. Kids, watch this!

Of course, it's not what we're supposed to be talking about, is it? So we'll have to abandon this interesting topic and talk about something more mundane: the SOLID principles of agile and object-oriented design. I can throw a whole bunch of other adjectives in there if you'd like.

## The Problem: Exponential Growth of Programmers

What goes wrong with software? How many of you have been a programmer for more than a year? Five years? Notice how that's half. Ten years? Notice how we've cut it in half again. Fifteen years? Half again. Why? That's interesting.

What is the population of programmers? How many programmers are in the world? It's a lot. It depends on if you count the VBA programmers, but it's a lot. It's probably close to 100 million, depending on how you count.

I started programming over 40 years ago. How many programmers were there in 1970? Well, no more than 50,000. It was probably more like 50,000, but it wasn't 100 million. If you go back 10 years before that, 1960, how many programmers were there in 1960? A couple hundred in the world. A couple hundred. Hardly any. They weren't even programmers. They were hardware developers that got roped into writing code because nobody knew what the hell this stuff was.

You think about this progression of time. We started with a few programmers in the '50s, managed to get a few hundred in the '60s, a few thousand in the '70s. Now we're close to 100 million. That is a geometric, exponential—you tell me the word—increase. It has a doubling rate. The doubling rate has to be five years. Every five years, the population of programmers doubles.

Which has an interesting implication: half the programmers have less than five years' experience. This has always been true. Old guys like me—people wonder, "How come there aren't more older programmers? They must all quit. They must all go chicken farming or something." We don't quit. There just weren't a lot of us. We're still all here. We're still writing code. It's just that there were hardly any of us to begin with.

How do we deal with the fact in our industry that we are stuck in an exponential curve that guarantees perpetual inexperience? Half of the programmers will have less than five years' experience, and that's going to remain true until that exponential curve tapers off, which probably will happen sometime in the next couple of years, maybe a couple of decades. But pretty soon. How do we deal with that? What is the problem with that?

The problem with that? Well, let me ask this question: how many of you old guys have been slowed down by really bad code? The young guys don't have their hands in the air. How many of the young guys have been slowed down by really bad code that you wrote yesterday? Yeah.

We know bad code slows us down. Why did we write it? Why do we write the code that slows us down? How much does it slow us down? A lot. Every time we touch the modules, we get slowed down again. Every time we touch the bad code, it slows us down.

Why do we write this stuff that slows us down? The answer to that is, "Well, we have to go fast." I will leave you to deal with the logical inconsistency there.

## The Key Principle: Go Slow to Go Fast

My message to you today is: you don't go fast by writing crap. You don't go fast by rushing. You don't go fast by tearing through the code and just making it work and releasing it as fast as you can. You want to go fast? You do a good job. Your grandmother told you this a long time ago. You want to go fast? You go well. You take your time. You study the problem. You move deliberately instead of rapidly.

A programmer who goes fast, a programmer who is rushing like crazy, is guaranteed to go slow. A programmer who sits carefully and thinks about it and types a few characters and looks at the code and cleans it a little bit and does this and that—he'll go fast. Doesn't matter if it's she or he.

Imagine this: you are having an out-of-body experience, floating above an operating table where a surgeon is performing open-heart surgery upon you. You're looking down at that surgeon as he is opening your chest and trying to repair the damage to your heart. How would you like this surgeon to behave? Carefully. Deliberately.

Now, he's got a deadline—literally. But you hope, even because of this deadline, even in the face of this deadline, you hope this surgeon is behaving carefully, calmly, deliberately, following his or her disciplines, going close to the book—not exactly by the book, but close to the book—calmly asking for an instrument, calmly doing a thing, acting like the surgeon knows what they're doing.

You don't want the surgeon acting like the typical software developer. Think about that for a while.

## Symptoms of Bad Software

What goes wrong with software? Well, that's what goes wrong with it. But we can be more specific. What are the symptoms of bad software? How do you know you have bad code? What does it do to you, other than the obvious thing of slowing you down? How does it slow you down?

It's confusing. Okay, fine. You look at it and you go, "Huh." Now, that's bad code. You look at the code and you go, "That's bad code," because good code should explain what it's doing. With good code, you should look at it and you should go, "Oh yeah. Oh yeah." It should be boring. You should read it and go, "Perfectly obvious." That's good code.

Bad code? Man. What else is it doing? What happens when you modify bad code? You break something. That's how you know it's bad code. If you modify something and that breaks something else, the code is bad.

### Rigidity

One of the symptoms is called **rigidity**. Rigidity is when you touch the code and you must now modify massive amounts of other code to come back into consistency with this modification you made.

Your boss comes to you one day and says, "Can you fix this bug?" You look at the bug report and you say, "Huh, I know exactly where this bug is." You don't tell your boss that. You just think to yourself, "I know exactly where this bug is. I actually put it in the code yesterday." So you tell your boss, "Three weeks," because that's the minimum estimate you're not allowed to give. An estimate less than three weeks? All the other developers come to your cubicle with clubs.

Boss goes away happy. He got the minimum estimate. He'll be back in three weeks. Now you go to the module because you know where this bug is. You go to the module. It's up on your screen. You look at it for a minute and say, "Yep, there's the bug. I know how to fix it. Let me do—" You know. "Wait a minute. If I do that, there's this other module over here that calls that function. I'd better go check it."

You bring that module up. "Oh man, I'm gonna have to fiddle with that a little bit. It changes a little bit. Oh wait, if I make that change, there's these modules over here..." And now you begin to chase the tail through the code. The weeks go by: two weeks, three weeks, four or five. Your boss comes back in and says, "I thought you were gonna be done with this in three weeks."

"I'll be done tomorrow, I swear!"

By the time you're done, you have touched every module in the system. Your boss says, "What the heck took so long?"

And you utter the immortal words of every software developer: "Well, it was a lot more complicated than I thought."

That is rigid code. Code that has dependencies that snake out in so many directions that you cannot make an isolated change without changing everything else around it. Bad dependencies. Systems that are coupled.

### Fragility

Another symptom of bad code, similar to that one, is called **fragility**. Fragility is the tendency of the code to break in many places, even when you only change it in one place. You make one very simple change, and a whole bunch of other things break. But they break in parts of the code that have no relationship to what you changed.

You made a little change here, and something over there doesn't work anymore. You make a change to the way the salary for hourly employees is calculated—just a little change—and now it won't print the report for the union boss. It crashes when it prints that report. Why'd that happen?

This is fragile code. Fragile code breaks in bizarre and strange ways, ways that you cannot predict. You make a change over here, something over there breaks. You didn't predict it. You don't know why it happened. You have to chase the bug around. Finally, you realize, "Oh yeah, that function over there set a flag, and that guy over there used that flag."

Your car has an electric window that doesn't work. You take it to a mechanic. Mechanic looks at it for a while, says, "Yeah, I can fix that." You come back the next day. Mechanic proudly shows you the electric window working. You thank him. You get in the car. You turn it on. The car won't start.

You're not going back to that mechanic. That mechanic's an idiot. That's what happens when managers and customers see software developers change one thing over here and something over there breaks. Nothing strikes more fear into the heart of a customer or a manager than that one, because all of a sudden, the managers and the customers who don't have any idea about the technology, they believe these guys have lost control.

They begin to think that if they looked under the hood, it'd be crap. They're right. They begin to suspect that maybe the quality of this product is not all that good at all. They become afraid of letting you make changes.

Has anybody worked at a company where the management finally said, "All right, nobody changes that module"? That module is too fragile. Every time you touch it, something else goes wrong. Nobody touches that module. That is the ultimate failure of a software developer, when the business, who is not technically competent, tells you you're not touching that module.

### Immobility

Third symptom of bad code: your boss comes to you and says, "You know that module you've got to write? You know, Joe, over in that other group over there, he wrote one just like it last year. You should go talk to Joe and see if you can use some of his code."

Oh, you know Joe, and you know the kind of code that Joe writes, and you don't want to go over there and talk to Joe. But your boss told you to do it. So you go over there and you look. Sure enough, Joe's module does just what you need it to do. But it does more than that. It couples to some bizarre framework. It uses some weird database. The more you look at it, the more you realize that yes, you could use Joe's code, but the problems you would be bringing in are so severe that you finally say, "It'd just be easier for me to write it myself."

The desirable parts of the code are so horribly coupled to the undesirable parts of the code that you cannot use the desirable parts of the code somewhere else. Joe's code was crap.

## The Root Cause: Coupling

How do we deal with this? What is the common thread of all of those flaws?

Well, okay, spaghetti is the word that we use to say, "Well, this code's all scrambled up." But there's a more technical term: **coupling**.

The reason the code is rigid is because the modules depend on each other in undesirable ways. The reason that the code is fragile is because the code depends on data structures in undesirable ways. The reason that I cannot take the desirable parts of Joe's code and use them in my system is because Joe's code depends on other code in undesirable ways.

The common thread there is coupling, dependency. It can be said, and I think accurately, that **the bulk of software design is managing dependencies**—figuring out where to put code and cutting the dependencies so that the dependencies don't run in strange and bizarre directions.

How do you do that?

## Understanding Source Code Dependencies

There are two people in the world I want to talk to: people who designed whiteboards. You know, chalk does not do this. We have space over here. Chalk also does not do that.

Let's say that I've got a main program. Let's say that we are writing code in C, circa 1971. I got a main program. My main program calls some high-level functions, high-level modules. Those high-level modules call middle-level modules. More than one. There's a bunch of these middle-level modules. The high-level ones call the mid-level ones, and the middle-level ones call lower-level modules.

You can see that there is this lovely tree of function calls. This is the procedure function call tree that every application has, no matter what the application is. Every application has one of these. We start at main, and we explode downwards towards the lowest-level functions in the system.

The flow of control goes that way. Main calls high-level. High-level calls mid-level. Mid-level calls low-level. The flow of control goes downwards.

Which of these modules knows about the other? In fact, let's make it simpler. I have a module M. I have another module N. Module M has a function named F. M calls F within N. Which of these modules knows about the other?

Flow of control goes that way. M knows about N. How do we know that M knows about N? If N changes, M has to... That must mean that the compiler knows. How does the compiler know that M depends on N?

There is a statement in the source code of M that says—we're doing C now—`#include n.h`. If it were Java, it would be `import n`. If it were .NET, it would be `using n`. But whatever language it is, the name N appears in M. There is a source code dependency from M to N, which I will draw with a red line.

Notice that the source code dependency (the red line) and the flow of control (the green line) point in the same direction. That's universal. In order for one module to call another, you have to have one module know about the other.

So how do the source code dependencies run here in this procedure calling tree? Well, they must go like this. The high-level modules know about the low-level modules.

Think about that for a minute. Wait a minute. The high-level modules know about the low-level modules. What rule does that violate? Inversion of... something?

Do you want your high-level policy polluted with low-level detail? This is what makes code hard to read. You're reading code, and you're trying to get the idea of what's happening at the high level, and all of a sudden you're dealing with string buffers, and you're down at some really low-level concept. We're trying to piece together what's really going on at the high level, but we can't, because our high-level modules depend on low-level modules, which depend on even lower-level modules.

Imagine some module down here which has an extremely low-level thing in it—God knows what—some low-level variable that we need to make a change to. I make a change to that module. Who recompiles? You have to follow the arrows back. Every module that depends on this low-level module is gonna have to recompile and be redeployed. A change to a detail affects high-level policy, and that's wrong.

If you think about that for any length of time, you think that's insane. I don't want details changing high-level policy. I want my high-level policy to be immune from details.

Why does code get rigid? Because we've got all this coupling down towards detail.

Why does code get fragile? Because I can make a little change down here and break a whole bunch of stuff up there.

Why is it that I can't reuse a bit of high-level policy? Because it's tightly coupled to all this low-level crap.

## Object-Oriented Design: The Solution

What's OO? What are objects? What is object-oriented design? What is OO? Why is OO part of every language that you work in nowadays?

Who's working in Java? Some people. Good. How about C#? Some more people. Ruby? Rails? Yeah. We're gonna talk about Rails. What else? What other languages do we have? Python? Yes. Ruby? Objective-C? Who's doing Objective-C, conquering the world with iPhone apps?

How old is Objective-C? When was it invented? 1980.

Who invented it? Brad Cox.

Why did he invent it? He was a Smalltalk programmer. Somebody made him program in C. He hated it. He wrote a little preprocessor in front of C, gave it some Smalltalk attributes, called it Objective-C. He started his own company, called it StepStone. He thought he was going to take over the world.

He did, for several years. This was the only option that C programmers had to do any kind of OO work. We all worked in Objective-C frantically from 1980 to 1985. Then Bjarne Stroustrup came along and published this lovely book called _The C++ Programming Language_. It looked so much like Kernighan and Ritchie that all the C programmers dropped whatever they were doing and immediately adopted C++. Poor old StepStone went out of business, and that language would be dead today were it not for an accident of history.

That accident of history was Steve Jobs, who decided he would hire a real businessman for Apple. Who did he go get? He got the CEO of Pepsi-Cola, John Sculley—as if the CEO of Pepsi-Cola knew anything about a high-tech computer company. What the CEO of Pepsi-Cola did know was enough politics to get Steve Jobs fired.

Steve Jobs went away with a lot of money. He had tons of money, so he didn't really care. He started a new company. The new company was called NeXT. This company sold hardware—black computers, very pretty. I have one in my basement.

He hired a bunch of programmers who were out looking for jobs because their language had disappeared from out from underneath them. They happened to be Objective-C programmers. They wrote the operating system for the NeXT machine. The operating system was called NeXTSTEP.

The company was a terrible failure. The machine never did anything. The software never went anywhere. It didn't matter, because Apple went back to Steve on their hands and knees: "Please come back, Steve. This guy's killing us."

Steve said, "Fine, but I'm bringing my team with me." Steve went back to Apple, taking all these Objective-C programmers with them, and they started working on the iPod. That's why you're programming in Objective-C—the worst possible language for an iPhone, by the way. You should not be working in that horrible language, but okay, that's the accident of history.

Why is it that all these languages are OO languages? It didn't used to be that way. We used to all work in Fortran or PL/1 or COBOL or C. Nobody knew about OO. How come all our languages are now OO languages? What's so good about OO that all our languages are now OO languages?

"Helps us organize code." "How?" "Encapsulation."

### The Three Pillars: Encapsulation, Inheritance, Polymorphism

Bum-bum-bum. Encapsulation. So, three magic words: **encapsulation**, **inheritance**, **polymorphism**. Those are the three magic words of OO. Every OO language has to be encapsulated, inherited, and polymorphic.

Did we have encapsulation when we were programming in C? Do we have any old C programmers in here? Okay, good, sir. Did we have encapsulation? We had **perfect** encapsulation in C.

All you had to do is forward-declare your functions and your data structures so you didn't have to implement them. You would forward-declare them in a header file, and then you would implement them in a C file. Your users would `#include` your header file. They could see nothing of your implementation. Perfect encapsulation. There was no way any of your users could see any of your data values. All they could see was your function signatures. They could see the names of your data structures but none of the members inside your data structures. Absolutely perfect encapsulation.

Objects completely screwed that up. C++ came along and put all the variables in the header file. What did that do? Wow! All the variables were suddenly visible to everybody. In order to get that under control, we had to invent these horrible, hacky words: `public`, `private`, `protected`. Terrible hacks. Ridiculous. Some kind of band-aid over the fact that we used to have perfect encapsulation, but now we've got this messy approach of trying to point at certain variables and say, "Well, that one's public, but that one's private, but that one's protected." Oh, and we'll invent other ones too that have no name.

OO did not give us encapsulation. OO weakened encapsulation. Every object language out there weakens the encapsulation that we used to have.

So when people tell you that OO is an encapsulated language, they are wrong. It destroyed good encapsulation and replaced it with this horrible, hacky thing of making the compiler check, "Oh, you're not allowed to touch that variable, so I'm not even gonna respond."

The encapsulation answer is just not there. We had encapsulation. Now we don't.

### Inheritance

Could you do inheritance in C? Yeah, but it was hard. Well, we had unions, and it wasn't too hard to take two data structures and give them common elements and change them only at the end, so the last few elements were different. Then you could cast pointers from one to the other and pass them around inside functions, just like polymorphic objects. We used to do this all the time. It's a very common approach in C.

C++ made it a little more convenient, but not a lot. Multiple inheritance was much more convenient in C++. That's why they took it away in Java.

Why does Java not have multiple inheritance? Wait, why does C# not have multiple inheritance? Because Java doesn't have it.

Why does Java not have multiple inheritance? "The diamond problem."

The diamond problem has been solved. The diamond problem is simple. You get basically a derivative of two base classes. Both those base classes have a common base class. This is the Deadly Diamond of Death. There's a huge ambiguity around that. We won't go into the details, but why didn't they resolve it? Because it had been resolved many times before by many other languages.

Why didn't Java resolve this? To simplify the compiler. This is my best guess. They were too lazy to deal with it, so they invented some horrible hack called `interface` and they stuck it in the language instead. Then we all bow and revere the interface. "Interfaces are good. Interfaces are wonderful."

An interface is nothing but an abstract class with a bunch of abstract methods in it. It's not some special thing. In fact, the only special thing in Java about an interface is what you can't do to it. You can't inherit multiple—you can't inherit multiple classes into another class. Why? Because the only thing you're allowed to multiply inherit is interfaces.

A hack. A lazy hack. They shouldn't have done that. They should have solved the problem instead of just throwing it out there. Then what happened? It went into a bunch of other languages.

You can tell I'm enjoying myself here, aren't you?

Okay. So we did have inheritance of a kind in C. We were able to simulate it, but it wasn't very convenient. So what I'll do here is I will give C half credit. I will give OO half credit for giving us a slightly more convenient inheritance.

Ruby programmers, how much do you use inheritance? Yeah. For what? Makes sense. Active Record? Do you use inheritance for polymorphism? Do you need to use inheritance for duck types? Don't know?

In Ruby, in Python, in dynamically typed languages, you can have polymorphism without any inheritance. In fact, all polymorphism in these languages comes without inheritance. Inheritance is not necessary. What do you use inheritance for in a dynamically typed language? It's to inherit behavior and variables, but not interface. You get all the polymorphism you want for free.

In Java and .NET and C++, we have to have inheritance to do polymorphism. One of the weaknesses of these languages is that you have to use inheritance to get polymorphism.

### Polymorphism

Which leaves us with the last of those three: polymorphism. Did we have polymorphism in C? Wow. Sort of. Sort of.

In C, we could do this: a copy application in C.

```c
while ((c = getchar()) != EOF) {
    putchar(c);
}
```

The UNIX copy program, vastly simplified. What does that program do? Oh, you did a nasty. It does, but not exactly. It copies from standard input to standard output. `getchar` reads a character from standard input. `putchar` writes a character to standard output.

What is standard input? It defaults to the keyboard, but it does not have to be the keyboard. It could be anything. `putchar` writes to standard output, which is usually the terminal—I use the word "terminal" shows how old I am—but it doesn't have to be the terminal. It could be anything.

Those are polymorphic calls. `getchar` and `putchar` are polymorphic method calls on a class named `file`, if you wish. They're just like any other virtual function in C++, just like any other polymorphic function in Java.

I have a type hierarchy there. Every I/O driver is just another type. I had perfect encapsulation in C. I had reasonable inheritance in C. I could get polymorphism in C.

How did that polymorphism work? How was it that the flow of control left `getchar` and got into the keyboard driver? How'd that happen?

No, it's much more clever than that. Much more insidious. Every I/O driver had to have five magical functions. Every I/O driver had these five functions. They all had the exact same signature: `read`, `write`, `open`, `close`, `seek`.

If you wrote an I/O driver, you had to implement those five functions, and the five functions had to have the same signature. You had `open`, `close`, `read`, `write`, `seek`.

The operating system would take pointers to those functions and put them into a table. When you called `getchar`, it would go to the table for standard input and say, "Where's the read function? Oh, I'll call that read function," not knowing what the read function was, just knowing that it had the right signature.

C++ programmers, what was that table? It's a **vtable**. Exactly. The way C++ implements virtual functions.

Java programmers, do you have pointers to functions? No. Why not? Because you have polymorphism. If you have polymorphism, you don't need pointers to functions.

C# programmers, do you have pointers to functions? You get those ugly delegate things, but they're not quite pointers to functions. No, you don't have pointers to functions.

C++ programmers do, but they don't use them if they're seen.

Ruby programmers, you have pointers to functions? Not really.

OO languages do not need pointers to functions because OO languages are polymorphic. If you have polymorphic dispatch, you do not need pointers to functions.

We did not do polymorphism in C very much. The reason we didn't do it in C very much was because it was dangerous as hell. If you created a table of function pointers, you had to load that table, and you had to make sure that everybody called the functions through that table. If anybody violated any of those rules, you had a terrible problem on your hands. So most of the time, we did not have polymorphism in C or in any language prior to C++.

When C++ came along, all of a sudden we got cheap, easy, safe polymorphism.

## The Power of Polymorphism: Dependency Inversion

Polymorphism is interesting because polymorphism allows you to do something with this that we couldn't do before. Let me draw it again.

Here's our module M. Here's our module N. It has a function named F. The flow of control will go from M to N, calling F.

But if I have polymorphism, if I have an OO language, I can do this: I can put an interface here. Yes, I know I'm using the `interface` keyword. My M module can mention the name of that interface and use it to call F. I will define F in here. The N module will derive from that interface.

Notice what has happened to the compile-time dependency. It points against the flow of control. This is what polymorphism gives you. Polymorphism gives you the ability to create one module calling another and yet have the compile-time dependency point against the flow of control instead of with the flow of control.

If you have that power, then you can take any of these red arrows here and turn any one of them around. You suddenly have absolute control over your dependency structure. If you have absolute control over your dependency structure, you can avoid writing fragile, rigid, and non-reusable modules. You can get around what goes wrong with software by carefully deciding which direction the arrows between modules should point.

That's what OO is. You may have heard that OO is modeling the real world. Nonsense. You may have heard that OO is closer to the way we think. These things were made up by marketing people in order to sell the idea to executives who didn't know what the programs were.

**OO is about managing dependencies by selectively reinverting dependencies in your architecture so that you can prevent rigidity, fragility, and non-reusability.**

## The SOLID Principles

There are several principles that we can talk about that take advantage of this aspect of polymorphism of OO. We're going to introduce a few of them here as long as we have time. Notice that I have not been using my slides, but everything I was talking about was actually there. Okay.

The principles we're going to talk about are called the **SOLID principles**. They're called the SOLID principles because the words happen to spell out the word SOLID, which is not something I knew to begin with, but someone pointed it out to me several years ago and said, "You know, this spells SOLID." So I quickly changed it because it sounded like a great word to use for a bunch of principles.

### S - Single Responsibility Principle (SRP)

The first of these principles is called the **Single Responsibility Principle**. The Single Responsibility Principle says this: **a class should have only one reason to change**. It should have a single responsibility to change.

Don't be confused about the word "responsibility." The word "responsibility" does not mean "function." Does not mean "what it does." The word "responsibility" means "change." It has one reason to change.

Look at that class up there, that `Employee` class. It has a number of methods on it: `calcPay`, `reportHours`, `writeEmployee`. `calcPay` calculates the pay of an employee. It's a payroll application. `reportHours` generates some horrible report that the auditors will read from time to time about the employees. `writeEmployee` is a database function that saves the employee to a database.

How many reasons does this class have to change? How many changes could be requested? How many sources of change are there?

Let's think about that. If `calcPay` cost the company a million dollars, which C-level executive would find out about it first? Would it be the CFO, the CEO, or the CTO?

It would be the CFO because he'd be the one looking at the books, in the accounts. Then he would go to the programmers and fire them.

If there were a bug in the `reportHours` function which cost the company a million dollars, which C-level executive would find out about it first? The CFO, the CEO, or the CTO?

The CEO, because he controls the auditors who check the reports about what got done and what got spent. He would go to the developers and fire them.

If the database crashed, destroying a million dollars worth of data assets because some programmers screwed up that `writeEmployee` method, which C-level executive would discover it first? The CTO.

So I have three different C-level executives interested in these three functions. If they don't work right, each of them will be upset for different reasons. If they want changes, if their particular organizations want change, those changes will come from beneath those particular C-levels.

A change to the database schema is going to come underneath the CTO. Some DBA is gonna want to change the schema, forcing you to change this class.

If the report needs to change, it'll be some auditor who wants to change. He reports to the CEO.

If the business rules for calculating pay change, it'll be some accountant who works for the CFO.

So this class has three different responsibilities. It has responsibilities to three different actors—three different people or three different organizations—all at the same time.

Is it possible for me to add a new feature to the `reportHours` function and accidentally break the `calcPay` function? It is.

"Well, I'm not going to do that. I'm not gonna put those three methods in my class because those three methods have different reasons to change. They change at different times for different reasons. They change based on the interest of different people in the organization. So I'm going to separate those three methods and keep them out of the single class."

You say, "Well, wait a minute. I thought this was objects. I thought objects were supposed to have the methods that make them work."

Yeah, sure, that's true, but you'd also better protect yourself from those C-level guys who are going to come and fire you.

So what do we do? How do we get those three methods into three different classes? Because I don't want them there. There's a whole bunch of ways to do that.

You could do it this way. You could create an `Employee` which has the `calcPay` method. You could create other classes like the `ReportWriter` and the `EmployeeRepository` that speak to the `Employee` method and manage to do their jobs. That's a possibility, although I don't much like that because these guys now depend on that, and if there were a change to `calcPay`, these guys would have to be changed. They would have to be recompiled and redeployed. I don't much care for that.

So maybe what we'll do is this: maybe what we'll do is we'll create an interface named `Employee` that has at least one of these methods, and then we will derive from that interface another one of these classes that implements `calcPay`. Maybe we could do that.

Or, gee, maybe we could have a single class that influences all three methods but has three different base classes, one with each of the methods. That would be interesting, although I don't know if it solves the problem.

Or maybe what we could do is create three classes, one with each method, and put a facade object out here that delegates to them.

There's a million ways to address this problem, a million ways to move around it. The important part is: **don't put functions that change for different reasons in the same class**.

What reason is the `reportHours` function going to change for? What reason will the `reportHours` function change? Yeah, they want to report fractional hours in a different way. They're not changing the substance of the report at all. They're just changing the format they use to report fractional hours. They used to report fractional hours with a decimal point. Now they're going to report fractional hours with two numbers—one for the number of hours, one for the number of tenths—without the dot anymore. They don't like the dot.

You think these guys don't do things like that? Yeah. "We don't like that dot." Okay.

So you work in there and you change the dot. You pull the dot out, and you break the `calcPay` method. You don't want methods in a class in the same class if they change for wildly different reasons.

I don't want to expose the `calcPay` method to the vagaries of the auditors who want to change the format of a report. Formatting should go in one place. Calculation should go in another. Database in another. If you've got some other concern, put it in another. Don't mix concerns in your classes.

### O - Open-Closed Principle (OCP)

The **Open-Closed Principle**—the O in SOLID—was invented, by the way, by Bertrand Meyer in 1988 in a lovely book called _Object-Oriented Software Construction_. If you haven't read that book, go read it. Read the first edition. The second edition is a thousand pages long. No one can possibly read it.

The Open-Closed Principle says this: **a class or a module should be open for extension but closed for modification**. What does that mean? It means you should be able to change what the module does without changing the module. You should be able to change the behavior of the module without changing the module. It should be open for extension so its behavior can be extended, but closed for modification.

Now, this sounds oxymoronic. It sounds impossible. How the heck can you change the behavior of a module if you can't change the module? But we do it all the time.

This module here, this `copy` module, I can change its behavior by writing a new I/O driver. I can write a new I/O driver that reads characters from the optical character reader, and my copy program can call that I/O driver. Do I have to recompile my copy program? No. I don't have to change it. I don't have to recompile it. I don't have to do anything to it, and yet it can call the optical character reader.

The optical character reader driver is written long after the copy program was compiled and put on my computer, but my program can still call it. I can extend the copy program because I have polymorphic calls within it, and I can implement those polymorphic calls anytime I want to in any interesting way I want to. I can extend the copy program.

If you can conform to this principle, then when you add a new feature to an application, you should be able to do that by writing new code and not changing any old code. You're not gonna modify it. You're going to extend it. So all your modules are closed for modification, but they can be extended, so you can extend your application with a new feature without changing any of the existing modules.

If you have conformed to that principle, is this possible? Yeah, that's possible. I can do it. Here, I'll show you. It's easy.

#### The Failing Version (Violates OCP)

We'll do this in C, or some language that sort of looks like C. I've got a simple application. By the way, this is the failing version. This is the version that does not conform to the Open-Closed Principle, written in C.

I've got some `shape.h` class. It has an enum in it. That enum has two enumerators: `CIRCLE` and `SQUARE`. There is a data structure within it called `shape` which has one data element, which is the enumeration.

I have another data structure called `circle`. It begins the same way that `shape` does, which means I can cast a `circle` to a `shape` and still use it. This is kind of fake inheritance here. I've got a `double radius` and a center point, and a function that can draw a circle.

I've got a data structure named `square`, in another file called `square.h`. It also begins with the enum data structure, has a `double side` and a `topLeft` point, and a function that draws the square.

Now I have this function here, `drawAllShapes`. `drawAllShapes` takes an array of shape pointers. It loops through the array of shape pointers. It asks each shape what its type is. If it's a square, it calls `drawSquare`. If it's a circle, it calls `drawCircle`.

What bad thing happens because I wrote the code this way? This violates the Open-Closed Principle. So if I add an oval, I must add another switch, another case to the switch statement. But that's not the first thing I did. What's the first line of code that changes?

The enum. And that's in `shape.h`. Who `#include`s `shape.h`? Everybody. Who recompiles? Everybody. `circle` recompiles because I added an oval. `square` recompiles because I added an oval. That's fundamentally sick. `square` and `circle` don't give a damn about oval, but they've got to recompile just because I added an oval. Something's really wrong with that.

That's rigid. That is the symptom of rigidity. You have to compile too much because of a change—compile things that should not have had to be recompiled because of a change.

But it's worse than that because I do have to modify the switch statement. I've got to go to the switch statement, and I've got to add the oval case. You think, "Well, that's not so bad. It's just a switch statement." Huh. But there's more than one switch statement. Switch statements are like lice. There's always more than one. Switch statements replicate and duplicate and spread through the system.

There are switch statements of this form for `drawAllShapes`, `eraseAllShapes`, `dragAllShapes`, `scaleAllShapes`, `rotateAllShapes`—anything you can do with shapes, there's a switch statement for it. Except that they're not all switch statements because some programmers don't like switch statements and they use if-else statements instead, but they're still switch statements.

Now what I have to do is I have to find them all, and I have to modify them all. I have to put oval in all of them. That may seem simple, except for the fact that programmers are strange beasts. They like to do logical optimizations. If they can get rid of a case or do some special ANDs and ORs or something, they will do that.

For example, what is the case of a circle in `rotateAllShapes`? Nothing. You don't do anything to a circle when you rotate it. It doesn't have a case.

So this is fragile. It's going to break because I'm not gonna find all of those switch statements, and I'm not going to logically decode them properly, and I will wind up creating some problem because I missed something.

This is rigid because it recompiles more than it ought to. It is fragile because I can't possibly find all the switch statements and modify them correctly.

But that is not the worst of the problems. The worst of the problem is this: our boss comes to us and says, "You know, we've had a lot of trouble with this startup company, and we're about to run out of funding. The sales model we had just isn't working, but we got a new idea. We hired a consultant, paid him a lot of money, and he told us what we had to do. What we have to do is this: we have to give away `drawAllShapes` for free. We're going to take that, we're going to put it in a JAR file or a DLL, we're going to put it up on our website, give it away for free. Then we're going to take all the circles and squares and ovals and triangles, and we're going to put them in separate JAR files or DLLs, and we'll sell them all for $5 each on our website."

We have to tell our boss that we can't do that because the switch statement has a dependency on `square` and `circle` and `oval` and `triangle`. We cannot separate them. We must deploy `circle` and `square` and `triangle` and `oval` with `drawAllShapes`. There's no way to independently deploy them.

We tell our boss, "Our architecture does not support what you want to do." Then we go out of business and look for other jobs. That is **immobility**. The reusable part was not properly decoupled from the goop.

#### The OO Version (Conforms to OCP)

How can we fix that? Are you ready for the "why"? Here comes the line. This is the OO version. I want you to think of soft music in the background, sound of wind blowing through the trees, fuzzy bunnies jumping over hillsides.

```cpp
class Shape {
    virtual void draw() = 0;
};

class Square : public Shape {
    void draw() override;
};

class Circle : public Shape {
    void draw() override;
};
```

And look at `drawAllShapes`:

```cpp
void drawAllShapes(Shape* shapes[], int count) {
    for (int i = 0; i < count; i++) {
        shapes[i]->draw();
    }
}
```

The elegance. The glory. The beauty of it all. They're all shapes. Loops through the shapes and just tells each one to draw itself.

Oh, God. What happens when we add the oval? What here must recompile? Nothing. Nothing. I can add the oval. Nothing here recompiles. Open-Closed Principle.

Is it fragile? It's not rigid. It can't be rigid because nothing has to recompile. Is it fragile? Can't be fragile because everything I can do to a shape I must implement in that shape. There's no way for me to forget to implement a method. It's not fragile. I don't have to go hunting for switch statements. There's no funny logical optimizations. The `rotate` function will be implemented to do at least something in `circle`, even if it's just open-close brace. It's not fragile.

But the best part is that when our boss comes to us and says, "Yeah, we want to put those shapes in one DLL and we want to put the draw shapes in another DLL," we can do that. Because `drawShapes` doesn't have any idea that squares and circles exist. The high-level policy here does not know about the low-level details.

The flow of control still works the same way. This loop calls these draw functions. But at compile time, the high-level policy does not know about the low-level details. None of the design flaws appear. None of the symptoms of bad code are evident.

#### The Lie

I said this was the lie. Why is this the lie? Well, it turns out that customers have a different idea in mind. This protects us from something. What does it protect us from? What changes does it protect us from?

New shapes. If we make new shapes, nothing here changes. But our customers have no interest in new shapes. What they want is for all the squares to be drawn first and all the circles to be drawn second. That's the change they want to make. They want all the circles to be on top of the squares, so they want the ordering to change.

Now, had we known that ahead of time, maybe we could have invented an abstraction that protected us against the ordering of the shapes. But we didn't know that ahead of time. We thought this was a perfectly good real-world model. Great! Squares derive from shape. Circle derives from shape. How better could you model the real world than that?

It's just that's not what the customers care about. Customers are very good at somehow knowing what your design is and then choosing the new feature that will completely thwart your design. This is a fact of life. It's one of the great flaws of OO design, because in order for OO design to protect you from the customer, you must know what the customer is going to do. Customers always do the other thing.

So you can't just model the real world and hope for the best because the customers will certainly find some interesting way to screw you.

#### The Pragmatic Approach

What we do instead is we take a very pragmatic view. We say, "All right, what we're gonna do is implement the simplest thing we possibly can. Then we're going to get it out in front of the customer as soon as we possibly can. We're gonna ask the customer to change it. Please change our code."

Have you ever done this? Please change our code. Give us new features. The customers will suggest new features, and this gives you a clue about where the axis of change really is in your application. Then you can implement the abstractions that protect you from those changes.

Do not think that you know, because you don't. Go out to the customers as early as you can. Find out what they're going to do to you, and then build the abstractions into your application that will protect you. They won't be what you think they are.

### L - Liskov Substitution Principle (LSP)

I think we have time for one more. This is called the **Liskov Substitution Principle**. It was invented by Barbara Liskov in 1988. 1988 was a good year for principles.

Barbara Liskov said this—well, actually, this is a paraphrase because what she wrote was a mathematical formula, but what I will do is paraphrase that formula: **derived classes must be usable through the base class interface without the need for the user to know the difference**.

That makes perfect sense, right? I've got some user, and the user uses a base class. But I've got a derivative. I should be able to substitute that derivative in, and the client shouldn't know anything about it. Simple polymorphism, right?

#### The Dreaded Rectangle-Square Problem

I have a `Rectangle`. In this class we wrote a long time ago, it's got a field named `height` and a field named `width`. They're both real numbers. There's a `setHeight` and a `setWidth` function.

A new requirement comes along. That new requirement is for a `Square`. Now, clearly, a square is a rectangle, so `Square` inherits from `Rectangle`. Makes perfect sense.

Wait a minute. How many fields does a `Rectangle` have? Two: `height` and `width`. How many fields does the `Square` need? One. How many will it inherit? Something's wrong. `Square` cannot inherit from `Rectangle` and only have one field.

Now, that means we're gonna waste memory. Memory's cheap. Let's just waste the memory and do this. Let's override `setWidth` in `Square` so that it sets both the height and the width. We'll override `setHeight` so that it sets both the height and the width. That solves the problem.

Imagine some user of `Rectangle`. That user calls `setHeight`. Does the user that calls `setHeight` on `Rectangle` have the right to expect that the width won't change? Yes.

But if I pass him a `Square`, he will call `setHeight`, and the width will change. That will corrupt his internal finite state machine, and he will then corrupt the heap. He will discover this a billion instructions later when you get a `NullPointerException`.

What happened? Oh, by the way, you're the guy writing the code that just blew up. You're the guy writing the code that's calling `setWidth` on the `Rectangle`. You get your logic analyzer out, and you walk through the back trace through a billion instructions. You find, after weeks of analysis, that the reason you blew up is because somebody passed you a `Square`.

So what code are you going to write in your module that will protect you from the possibility that the `Rectangle` you're holding might actually be a `Square`? `if (rectangle instanceof Square)...`

By doing so, you will hang a dependency on the `Square`, something that you never wanted to know about. But your module, which was supposed to know about rectangles, will suddenly have a dependency upon the `Square`, violating the Open-Closed Principle and making you rigid and making you fragile.

Does anybody have any code like that in their system where you ask what type an object is? What do we use in Java? We use `instanceof`. In .NET, we use `is` or `as`. There's a bunch of different ways to do that.

It is not true that every time that you do that it's a problem, but most of the time when you do that, it is a problem. Most of the time. This particular interesting relationship causes that to happen.

#### What Went Wrong?

So what went wrong? Isn't a square a rectangle? A square is a rectangle, so that should be the right relationship.

Now, because that's not a square, that's a piece of code. That is not a rectangle. It is a piece of code. It is true that a square is a rectangle, but the class `Square` is not a square. The class `Rectangle` is not a rectangle.

If the class `Square` represents the square, the class `Rectangle` represents a rectangle. But **the representatives of things do not share the relationships of the things they represent**.

If anybody here has been divorced, you may have had two lawyers representing you and your spouse. Those two lawyers were probably not themselves getting divorced. The things that represent do not share the relationships of the things they represent.

You cannot simply say, "Well, that class has the name of something, and therefore I'm going to use an inheritance relationship." You have to understand what these relationships are.

We call that relationship "is-a." That's the wrong name for that. It's not "is-a." It got inherited from the artificial intelligence people in the '80s. They were doing interesting data structures called knowledge nets, and they had interesting relationships: "has-a," "tastes-like," "smells-like," "is-a." They all got fired because their funding dried up because artificial intelligence didn't actually do anything useful. So then they all became OO programmers, and they carried their interesting abbreviations with them. That's how we got the "has-a" and "uses" and "is-a" relationships. They named these relationships that way, but they're lies. They're not correct.

The inheritance relationship is not the "is-a" relationship. The inheritance relationship really is just the redeclaration of functions and variables in a subscope.

#### How to Fix It

How can we fix this problem? What is the relationship between `Square` and `Rectangle`? They do... but yeah, you'd want like "behaves like" a thing. What am I going to do? Am I going to have any relationship between these two classes at all? They might both derive from a common base class. Maybe the base class is `Shape`. But these two classes really have nothing in common at all. They don't have the same number of variables. They don't have the same behavior. They are not related, at least not as siblings, and certainly `Rectangle` is not the parent of `Square`.

This is a violation of the Liskov Substitution Principle because `Square` is not substitutable for `Rectangle`, even though it seems to fit the definition of "is-a." It does not fit the substitution rule.

Whenever you violate the substitution rule, you will eventually add an `if` statement that checks the type of the object to prevent you from crashing your system, which makes... this is about 8 o'clock.

Oh, you want to continue? You want a method `isRegular`? You want that up in `Rectangle`? And what you're saying is, "Is it a regular rectangle?" So I don't even want the `Square` class. I just want a regular... Now what would happen if I were then to set the height of a regular rectangle? Would it set the width? Would I modify the width? Would I dare? Or would I just set the `isRegular` flag to false?

If they happen to be the same, do I automatically set the `isRegular` flag? I calculate it. So `isRegular` will return true if the height and width are the same. Okay.

They're reals. What can you not do with a real number? You cannot compare them equal. Can I compare real numbers equal? People go to jail for doing that, by the way, especially if the real number represents money. Do not compare real numbers equal in a computer. Floating-point numbers can be greater than or less, but they are never equal. If you use the equal operator and it says they're equal, it's a lie.

I cannot compare them equal. You can compare them close, but you can't compare them equal.

## Conclusion

All right, I think we're done for the day. Thank you all for your attention. If there's any questions, I'm gonna hang around for a little while, but the rest of you can go.
