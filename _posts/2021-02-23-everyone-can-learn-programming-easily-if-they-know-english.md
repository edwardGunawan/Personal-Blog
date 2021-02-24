---
layout: post
title: Everyone Can Learn Programming Easily - If they Know English
date: 2021-02-23 23:13
summary: Some barriers for the foreign speaker in getting into the programming and what we can do to help
categories: ideas life-lesson
tags: ideas life-lesson
---

![Photo by Les Anderson](https://images.unsplash.com/photo-1488254491307-10ca8fa174c8?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1062&q=80)

With technology disrupting industry after industry, people from around the world are learning computer programming. The vision on the first worldwide web was to share knowledge and build trust globally on the internet. With the help of Google and other applications, we can all learn how to write programs through the internet. 

While the internet helps create a protocol and a platform for ease of sharing knowledge - most programming languages, documentation materials, and online instructional learning are in English. 

This makes it a barrier for non-native English speakers to learn about computer programming and share their knowledge worldwide.

Imagine if the most popular programming language you are using right now to create your website, HTML, is not English but Chinese. When you are about to inspect a beautiful website by inspecting its source code online - it is all in Kanji. Instead of having a `<title>` tag, it will have `<稱號>` tag. Instead of having `<article>`, it will have `<文章>`. You still feel motivated and want to tinker with the web. Therefore, you look up material online. However, you realized that the answer to the most popular programming community's questions is also in Chinese. You feel frustrated and drop learning HTML and learn Kanji first. 

In theory, the computer doesn't care about what language we are coding in as long as those languages can compile to a 0's and 1's byte code that the machine can understand. The `if` statement, `while` statement and the `for` statement help humans easily project computer instructions so that the compiler can translate those into byte code. 

But only some of us know all of those commands - those who speak English. 95% of the world population out there doesn't have English as their native language. The open-source learning course material out there in the world is in English. Most of the popular programming languages use their example in English.  Even searching stack-overflow, a popular programming community,  needs to be in English. Although you can use Google Translate to translate each sentence, the translation quality can be misleading or confusing. It has caused an enormous barrier for non-English Native speakers to access programming knowledge and express their intent to the Technology Community.

This article wants to share that non-native English speakers have two barriers to learning computer programming. After, I will mention three solutions that we can do as a software engineer to create a better community that invite non-native English speakers to easily navigate and broaden access to digital literacy and job opportunities from people around the world.

## Documentation is Mostly in the English Language
For better or worse, we must agree that English is the dominant language used in science and technology. The majority of the popular programming languages and their documentation are in English. Although several translations are available in some widespread documentation, it usually confused the learners even more to translate deep technical terminology.

Study shows that communication with general reading comprehensive significantly affects a person ability to grasp new knowledge - they have to use half of their brainpower to interpret one English sentence and the other brainpower to learn the new terms of programming languages [4].

Non-native English speakers who wish to express their programming ideas as documentation also have difficulty describing them. For instance, The creator of Redis, Salvatore Sanfilippo, talks about his struggle in expressing thoughts in English. They were about to run a TCP/IP attack, but they cannot write a post about it in English because he already put in 50% of his energy to understand the sentence he writes [2].

I remember catching up with my childhood friend, a computer science major in a Taiwanese Local University. He said that everyone who needs to learn Programming would need to have an intermediary English because it can be challenging to keep up with courses not taught in their mother language.


## They have a Hard Time Reading and Writing Code
We encourage developers to write readable and maintainable code. 

However, this readable code mandates one to create code identifier, variable, functions, class name in English. Most programming language keywords, such as `if`, `else`, `while` are all based in English. You can say that technical English-speaking language doesn't have a hard time learning most programming languages than English speakers because most of its syntax is derived from mathematics. You can say that most of the keywords - `if`, `else`, `while`, `type def`, "Class", "type" are not precisely what they mean in English. These "variables" all have specific meanings in programming. Nevertheless, other non-native English speakers struggle to understand what a `while`, class, or `instances` mean because those keywords might hold more significant meanings in their native language. For example, `while` in Portuguese has more meaning than what is used in the code [4]. 

Beyond the standard programming language keywords, non-native English speakers have a hard time reading abbreviated function names - something like `getCh()` or `strstr()` in C is hard to understand. 

Some languages don't abide by the subject-verb relationship. In Japanese, the grammar is Subject-Object-Verb. Therefore, they have difficulty writing a getter and setter because the grammatical components are different from Japanese. Also, lots of behavior-driven test scenarios mimic full English sentences to test specific behavior in the application. For instance,  if you want to write a unit test in Scala, a `Matcher` class lets you construct test cases like writing a sentence in English. 

<script src="https://gist.github.com/edwardGunawan/1724d759f1b58679dd1f8e2d580f6799.js"></script>


In terms of writing code, I found that most foreign speakers do an excellent job at naming _things_ in English, but they struggle with calling the behavior of its function. Some functions may have a meaningless name - something that doesn't make sense in English. Some function names may even be misleading - which is dangerous - because naming things in their native tongue is different from calling stuff in English. 

Since the first worldwide web and all the protocol are written in alphabetical letters and English words, does that mean that every person who wants to learn how to code needs to learn English first?

Possibly. Learning English will help you understand technical terms easier. However, there are also ways to help others understand your library or source code much more comfortably.

## Using Complete Variable Name and Avoid Any Slang from your Culture
We need to write our documentation to be as culturally agnostic as possible. 

Naming functions should not be abbreviated. Having abbreviated words such as `numList` or `getCh` can make things very confusing for non-native English Speakers. Because this language doesn't express any other behavior through its syntax, a developer needs to understand the function's name to know the intent of the procedure. Without knowing the function's name, you can only _guess_ what the function output will be. Therefore, the best is to describe your function's name in a full sentence, such as `getCharacter` or `listOfNumber`.

Some programming languages, such as Clojure or Haskell, have opt-out naming variables and name them `f` or `g` because its function definition has described what it does. 

Some of the variable naming conventions in Clojure API  have particular functional programming technical terms and mathematical terms function names like `reify` or `transducer` not to steer any misunderstanding to its client. 

One of the powerful tools of functional programming languages is that their syntax is like a mathematical formula. You can identify what the function is doing without looking at the name of the procedure. For instance, if we want to do some combine on a list in an element. We can write something like this: `def f[A:Monoid](list:List[A]): A`, which tells, "If I put a List of A as an input, the function will return A." Of course, having to say `f` is very general, and it is not suitable for the use case of summing a list. However, the function definition itself and a simple comment can help non-native English speakers know what to expect of `f` and how they can implement them.

## More Example and Less Text
Documentation should show more examples than text. Having any American celebrities joke or TV shows in the programming documentation to make reading the documentation much more manageable doesn't help the audience. 

More audience in the world doesn't know about the American celebrities or any jokes in Silicon Valley (an American TV Show). Therefore, it will help explain how to deal with issues when clients use the library—explaining the meaning of the error messages and why they occur in the first place—further, giving your client guidance on resolving those issues when they encounter through example code. 

For instance, most libraries will have a "get started" guide with a simple example of using the library. Explaining the philosophy behind designing the library and all the other specific functions at first can make the client even more confused. 

If you explain specific keywords that are hard to understand, an inline dictionary that explains the meaning of that words may help [4] your readers understand the unfamiliar terms while reading the context. For instance, _reify_ should have an inline dictionary specification of what it means in the mathematical context. In this case, reify is a computer programming terminology representing the type's abstract concept during runtime. 

You can also give an external link on that concept so that your readers can search on that terms by clicking the external link.

## Use more Illustration and Visualization in Technical Instructional Materials
One study shows that the easiest way to ensure learners store their information in their long-term memory is to pair the course material with some form of visualization [1].  Words only retain in our short-term memory, and we can only keep 7 bits of information. An image helps us move that information right away to long-term memory.

Visualization can quickly transcend language. For instance, understand how recursion in Java works can be challenging without any visualization and step-by-step walk-through. With proper drawing, we can illustrate how each stack of the function behaves on each recursive call- and how each stack frame behaves when the process is returned can help learners visualize and trace a recursive function.

Illustration and visualization images are helpful for library authors to illustrate their design philosophy. For instance, explaining the definition and the purpose of React DOM  are much easier to visualize in the image than in text.

## Conclusion
Programming is hard for us who knows English well. Programming is even more challenging for those people how don't know English at all. 

One reason why programming is harder for non-native English speakers is that all the relevant material to learn about programming is not in their native language. 

When writing or reading source code, they have a hard time constructing a readable code because they cannot name the function as expressive as native English coders. 

Nevertheless, we can create a smoother experience for non-native English speakers to learn programming by creating a more inclusive language in our code, documentation, and community. Besides, having a more visual image and videos to illustrate a coding concept can quickly help others learn about the idea. 

Knowing these problems helps us write more language-inclusive documentation and be more patient and empathetic in sharing knowledge with non-native English speakers. 

Suppose we want to make programming become a universal tool for expressing ideas and thoughts. In that case, we must care about what we mentioned in a regular blog post and our documentation, our functions, and our codes to make it universally available for everyone. In other words, our computer has to learn the universal language: empathy.

## Reference
[1] Admin, Admin. Studies Confirm the Power of Visuals to Engage Your Audience in ELearning, 9 July 2014, www.shiftelearning.com/blog/bid/350326/studies-confirm-the-power-of-visuals-in-elearning.
[2] Chistyakov, Artem. The Language of Programming, 28 June 2017, temochka.com/blog/posts/2017/06/28/the-language-of-programming.html.
[3] McCulloch, Gretchen. "Coding Is for Everyone-as Long as You Speak English." Wired, Conde Nast, 4 Aug. 2019, 10:00 AM, www.wired.com/story/coding-is-for-everyoneas-long-as-you-speak-english/#:~:text=In%20addition%20to%20these%20four,System%20(Bengali%2C%20Gujarati%2C%20and.
[4] Philip J. Guo. Non-Native English Speakers Learning Computer Programming: Barriers, Desires, and Design Opportunities. Retrieved from https://pg.ucsd.edu/publications/non-native-english-speakers-learning-programming_CHI-2018.pdf
