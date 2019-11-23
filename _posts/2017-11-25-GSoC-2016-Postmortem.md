---
layout: post
title: "(GSoC 2016 Postmortem) Pride and Prejudice"
subtitle: "Lessons learned from failing to meet every deadline despite 60-hour weeks"
date: 2017-04-01
url: antlr4-cpp-cmake.html
---


### NOTE: This post was made around mid-August 2016, when the GSoC was about to end but the results were not in yet

GSoC is nearly over and whatever the result may be, I believe it's worth to make a brief self-reflection. After all, GSoC is first and foremost about learning, and as such it would be a waste to not take as much value as possible from all those hours in front of the computer. And I did spend them, trust me. Here's a brief summary of the major work periods that underwent this summer:

__From start until the first midterm__
This period was easily the most stressful of my life, and took a high toll on my body and mind. In my University June is reserved as the exam period, which in reality translates to the "juggle final projects and preparing final exams for two insane weeks, then just crash and sleep for a week" period. This year however, the exams were scattered through the entire month, with roughly a week between exams and a couple of final projects to deliver in between. This seemed extremely good, but the reality proved different.

On the other hand, the organization had perfectly reasonable concerns that I would show up to the midterm empty-handed, so there was pressure to have _something playable_ by the midterm.

Those two factors led to a situation where every ounce of time and energy _had_ to be allocated to either a project, and exam, or GSoC. For a month, my routine consisted of waking up reasonably early, spend 5 hours doing schoolwork, eat, and then just work on the GSoC until I just couldn't think straight, which could be anywhere between 11pm up to 1am the next day. I can't recall correctly, but I think I took one or two afternoons (not whole days, mind you) off in June.

Needless to say that my school results were less than stellar, albeit not as bad as I thought they would be (about 1.5 point drop from the mean), and the GSoC code suffered, but more on that later.

__Going home, and the rude awakening__
After the schoolyear was over, I returned to my parent's house to spend the holidays with them, and that was the bulk of the summer. I was extremely exhausted from the entire schoolyear and the devilish month of June, so this felt like a slow crawl of death more often than not, but it was much more in tune with what I pictured GSoC would be. Giving up most of the relax time for hard work making something great. Through grunts and a couple of mice broken in frustration in front of GDB and Git, I manged to slowly add functionality until the game was winnable (a very emotional moment, actually).

Looking back, I wish I had just taken a week off after the exams and the midterm. I feel that the resting time would have made me extremely more productive in the long term. I exhausted all my reserves in June, and I should have taken the time to reload, but hindsight's 20/20 I guess.

__The final sprint__
Now I am in the final sprint, wherein I submit the PR with the engine, and just dedicate myself to fixing bugs and making the code as good as I can. There will be no more functionality added to the engine in the GSoC context, and that delimitation makes those 8,500 lines look more manageable than ever.

I am taking it much easier now, partly because I've been sick and partly because there is not much else to do _within GSoC_ (the engine itself is far from complete), but at the same time I really want it to be over, and finally have some guilt-free free time. I feel satisfied with the effort (not so much with the results), and I can now count GSoC amongst the experiences that shaped me as a programmer.

### The green list, in which I bask in the things I did right from the beginning, thereby proving that I wasn't totally worthless to begin with

- __The communication with my mentor:__ I had the luck of having a really awesome mentor to guide me, without whom I probably would have been out of the program around June 21st. From the very beginning we decided to have as much communication as possible, even if things went wrong. We used great tools like GitHub, Slack and Trello to make it easy for him to know exactly how the project was going at a glance, anytime, anywhere. That meant that whenever I struggled (and ultimately failed) to meet a deadline, he knew exactly why that happened, how much (or how little) work I put into it, and could instantly assess the situation. I firmly believe that if I succeed at completing this program, it will be due to this open, complete and sincere communication.

- __Days off:__ During the post-midterm period, I began to actually think about my work habits, and I found my personal preferred balance for this type of all-consuming projects. I really prefer to not take weekends off, but rather "reserve" those days off for when I truly need them. For example, if I'm working on just playing the game looking for bugs for a couple of days, there is very little to be gained from taking Saturday off, as I'll most likely _not_ be burnt out by playing the game (as old-school as it might be). Rather, I find it is better for me to have the flexibility to just quit for a day and a half, with a clear conscience, if I get extremely frustrated finding a bug. Often I found that bugs were much easier to find after a whole afternoon with my girlfriend or playing Starcraft. Programming requires mental energy, so managing that energy wisely is vital for optimizing the results.

- __External support network:__ I just couldn't have done the insane amount of hours without my family and girlfriend being so supportive and understanding, I owe that opportunity to them.

### The red wall of text, wherein I explain what I wish I had known from the start.

There were a number of things that went horribly wrong, and that I feel prevented me from meeting the deadlines. They are all mistakes on __my__ part, so apologies if anything comes across as blame-shifting, it is not my intention at all.

The gravest mistake I made was __overpromising__ in June. I promised that I would try to have a playable engine working by the midterm, at a time where I just wasn't conscious of the sheer amount of work it would be. I thought I could just _power through it_ (hence the prejudice on the title), but with the schoolwork and everything I just didn't have enough energy to give to it. As a result, I rushed every single piece of code, didn't even bother to find a way to write tests (which later proved dead-simple), and made a big monolithic mess. The original engine was written in JavaScript, and boy-oh-boy does that language love global variables and functions. I was used to writing small classes (no more than 100 lines, for the most part), and when I translated the JS code into C++ I forgot all my experience and just crammed everything into 3 big classes. Most of the time I didn't even stop to make sure I understood the functionality, I just mindlessly translated the code and violated encapsulation _just to get it working_.

Guess how that went. The result were far too many late nights in GDB trying to hunt an elusive bug that manifested itself in three different classes at once. My mentor can tell you that, at this stage of development, I just hated my code and didn't want to show it to anybody.

Now this is a learning post, so this complaining would be of no use if I didn't propose a solution. Well, I don't have one, but Uncle Bob Martin does:

[Great advice](https://youtu.be/zwtg7lIMUaQ?t=56m23s)

This is not to say that it was wrong from the organization to ask me to deliver something, but rather that it was wrong of me to sacrifice my professionalism and everything I knew to be _the right way_ in lieu of meeting the deadline. I am confident that if I had followed Uncle Bob's advice, and I stuck with what I knew, I would have had a higher chance of meeting the deadline. If you want to do it quickly, do it right.

Now, that was the biggest blunder, but what about the rest? Here are some others:

__Not reading up on the codebase before GSoC:__ My first task in the program was to extract a GUI system that a member of the organization had wrote to emulate a Macintosh desktop, and add the functionality to allow custom borders with it.

Now that task is simple enough, it only requires moving things from place to place and establishing new links between them. However, I'd say that the task complicates significantly when you have know idea about any of the code in that codebase works. As an example, I didn't even know what the word _blit_ meant, something that is fairly important when you are working with surfaces and graphics. As a result, I spent far too much time with that simple task.

Now, this all could have been avoided if I had just opened the codebase _before_ the program. There were legitimate reasons not to do it, namely that I wanted to finish as many school projects as possible before GSoC, and that I wouldnt even know where to look, but I feel that I could have spent an extra hour every day looking at the code to avoid this situation.

__Not asking questions:__ During the aforementioned initial bump, I asked many questions via IRC that in hindsight had easy answers. That gave me the impression that I was asking too many questions, and after taking to a memeber of the organization I decided that I would try to look for answers by myself _every time_ I had a doubt. I was learning "proper C++" _at the same time_ as I was doing GSoC, so I had a nontrivial amount of questions.

So until close to the time I submitted the first PR, I did not ask a single question on the IRC, and in turn spent hours and/or days looking for answers myself. That was obviously not efficient, and what I should have done was to _set the pride aside_ (hence the title of the post), and just admit that I don't know something, and have someone more knowledgeable than me point out the super-obvious mistake in about 5 minutes total.

Ego is a bad thing (TM).

__Not being careful and crafty:__ Especially with formatting and leftover code, I realized that I had been very sloppy. I changed IDE's mid-program, and didn't even give any thought to formatting until the PR. This was wrong, very wrong, not only because it was annoying and embarrassing to have about 30 minor formatting and leftover code flaws pointed out, but also because it's not only _my_ time that's being wasted hunting for formatting errors, it's _some other person's time_. That was downright disrespectful on my part. I am profoundly sorry for the time I made other people waste because I just didn't clean up after me.

These are the most flagrant mistakes I can point out. I am sure there are more (maybe comment? All feedback is welcome), so I will write another post if I deem it worthy.

Until then, happy coding.
