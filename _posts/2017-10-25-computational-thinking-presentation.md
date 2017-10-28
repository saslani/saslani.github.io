---
layout: post
title:  "Computational Thinking - SCNA 2017"
date:   2017-10-20 22:04:39
categories: computer science
---

I've been an engineer for the past 14 years, but probably have been only thinking computationally for the last 7. To explain what I mean by that, let's start with talking about what computational thinking is NOT.

![Computational Thinking for kids of all ages](/assets/images/scna_extended/01.jpeg#scna_extended #scna_extended "Computational Thinking for kids of all ages")

___

Computational thinking is NOT thinking in programming language syntax. I think the worst thing you could do to get someone interested in programming is to introduce them to syntax like Java, C, or Fortran on day one. Why would they want to bother learning these unfamiliar, arcane details like pointers, classes, compilation? Rather, let's start with a goal. What problem are they trying to solve? Can this problem be framed as a computational one that a computer can do much faster than we could?

    #include <iostream>

    using namespace std;

    struct node
    {
      int data;
      node *next;
    };



    class linked_list
    {
      private:
        node *head,*tail;
      public:
        linked_list()
        {
          head = NULL;
          tail = NULL;
        }

        void add_node(int n)
        {
            node *tmp = new node;
            tmp->data = n;
            tmp->next = NULL;

            if(head == NULL)
            {
                head = tmp;
                tail = tmp;
            }
            else
            {
                tail->next = tmp;
                tail = tail->next;
            }
          }
        };

___

Nowadays, there are over 256 programing languages. The first program I can remember writing was over 20 years ago in DOS. When I started programming professionally, 14 years ago, I always gave it my all to deliver the project. But it took me some time to really go **deep** to try to understand **why** the program was going to work before I even wrote it.

![](/assets/images/scna_extended/03.jpeg#scna_extended )

___

In 2010, I left ThoughtWorks and started my own consultancy, Tested Minds. Which then allowed me to really think and pick projects that I was really interested in. Then, I made my first project from a genuine desire to have a computer solve a problem for me. I wanted to teach my husband read and write Farsi. I wanted him to practice as much as he wanted too. But I didn’t have time or was available all the time to sit with him.

![](/assets/images/scna_extended/04.jpeg#scna_extended )

___

The computation I wanted literately to do included things like:

* Provide a user with immediate feedback about whether they'd recognized the character.
* Show aggregated stats about a user's progress.

My mother is a retired elementary teacher, and most of her career she was in first grade. She had performed these computations for a long time! Her and I put a curriculum together which I then put it online for free as literately-Farsi!

![](/assets/images/scna_extended/05.jpeg#scna_extended )

___

There’s this famous quote:

> "Computer science is no more about computers than astronomy is about telescopes."

Sure, I agree with it to some level, but learning about astronomy without **ever** using a telescope, I'd wager, would be **really frigging hard**. Learning to think computationally **without ever** writing code or using computers, similarly, holds back a learner. I think the question is really about **when** to introduce code, not if.

So this raises two questions:

* How early can we introduce computational thinking?
* How early can we introduce programming?

![](/assets/images/scna_extended/06.jpeg#scna_extended )

___

Let’s look at computation vs arithmetic

The word 'computation' is often used synonymously to mean 'numeric computation’. Computation is the act of processing the data, calculation and reckoning

Arithmetic is type of mathematics, integers, real numbers, complex numbers. You can apply addition, subtraction, multiplication, and division to those numbers

Mathematics is the way to precisely represent data and proof a formula. In order to do computation and order the computers to be able to run algorithm you need to be quite precise, that’s why mathematics is important in computation

![](/assets/images/scna_extended/07.jpeg#scna_extended )

___

Let’s look at the Fibonacci Sequence example:

Without using any “computational devices”, most of us will slow down around the 10th digit. Let’s see this in action, try this along with me. What comes next in the sequence?

So we can already see that we have already started to slow down…

Of course some of you geniuses can go easily up to 15…20…30…digits. But you’re weird…in a good way! :)

![](/assets/images/scna_extended/08.jpeg#scna_extended )

___

How about this one?
How long would it take a human brain to get this answer? And how likely would we be to make a mistake along the way because of a bug in our brain’s programming?

    What's the 100th number in the Fibonacci sequence

___

This is where a computer can really help us out - Clojure syntax.

How long does it take for us to write that in our language of choice once you know an algorithm to generate the sequence?
How long would it take for a computer to run that code?

    (def fib
      (->> [0N 1N]
        (iterate (fn [[a b]] [b (+' a b)]))
        (map first)))

___

    (def fib
      (->> [0N 1N]
        (iterate (fn [[a b]] [b (+' a b)]))
        (map first)))

    (nth fib 100)

    **345,224,848,179,261,915,075**

    "Elapsed time: 0.059953 msecs"

___

Before we write the code, we need to frame the problem in a precise, repeatable recipe, **an algorithm**. What needs to happen in between the inputs we have and the output we want? How will these inputs and outputs be modeled. What data structures will we need for that modeling?

I’ll make the claim that there’s nothing the computer can do that the human brain cannot, given enough time and memory. But are the computers always faster and more efficient?


    ;; Define the Fibonacci as:
    (def fib

      ;; start with 0 and 1 as a vector for initial inputs, a and b.
      (->> [0N 1N]

        ;; create an infinite, lazy sequence of vectors by iterating a function over those inputs that returns a new vector of b and the sum of a and b
        (iterate (fn [[a b]] [b (+' a b)]))

        ;; map the first function over that sequence to convert to an infinite, lazy sequence of BigInt’s.
        (map first)))

    ;; We can now use functions like nth or take on this sequence.
    (nth fib 100)

___

Remember captcha? we all love it so much!

Noisy image recognition is a computational problem that humans can do subconsciously, but that computers struggle with. We could have a whole talk about the work going on with artificial neural networks to solve this type of problem, but what if we just put a human brain to work?

And that’s what Luis Von Ahn did!

![](/assets/images/scna_extended/13.jpeg#scna_extended )

___

How about Amazon Mechanical Turk: https://www.mturk.com/mturk/welcome

This is an example of what our brain can read and comprehend with one look but computers are not yet sophisticated enough to do. Most humans can't do numeric manipulation quickly. Most computers can't do free-form pattern-matching the way our brains can.

![](/assets/images/scna_extended/14.jpeg#scna_extended )

___

This is Grady Booch, he’s a software engineer and some of you might know him as one of the guys behind UML… but he’s spending much of his time these days in computation and AI.

I love this quote from him:

> "Story of computing is the story of humanity"

![](/assets/images/scna_extended/15.jpeg#scna_extended )

___

Computing started out as human computers (mostly women). Computation, preformed mostly by women, in WWII was one of the key factors we won the war!

In one of his articles, Grady mentions. Computational thinking drove our imagination, it goes beyond what computers can do and what we can do in present time

![](/assets/images/scna_extended/16.jpeg#scna_extended )

___

We are still the most extraordinary computers of all.

You are computing on a daily basis. You planned your route to this conference. You performed a scheduling algorithm to arrive here on time. You just might not have done any of this consciously.

How do we gain skill in **consciously** thinking of problems computationally? And if we can do so as adults, how early can humans start? Can we make computational and mathematical thinking like a second language at home to teach our kids while we practice ourselves?


![](/assets/images/scna_extended/17.jpeg#scna_extended )

___

Well, as a mother of two, I have a couple of research subjects that volunteer for all of my experiments!

* How do I prepare my children to be excited about problem solving in their world?
* How do I teach them to look at numbers and analyze the facts…not just listen to other people’s opinions?
* How I prepare them to take a complex problem in life and break it down till they find a solution?

My bet is that computational thinking is a great way to start.

![](/assets/images/scna_extended/18.jpeg#scna_extended )

___

Now let’s talk research ethics here for a second.

Before you accuse me of being a mad scientist and treating my kids like minions, let me show you some of the homework I have done.

When we were expecting our first child six years ago, we were complete apprentices in parenthood. So my husband and I started reading a lot of books and watching a lot of documentaries about early childhood education and infant brain development.

At just a few months old, babies can…

![](/assets/images/scna_extended/19.jpeg#scna_extended )

___

Predict sequences of events:
* running water => bath time
* mommy goes to fridge with a glass => water
* putting shoes on => go outside

![](/assets/images/scna_extended/20.jpeg#scna_extended )

___

Connect cause and effect:
* shaking a rattle => makes a sound
* pushing a toy off the table => drop, bang…the joy of gravity!
* cry => Mommy or Daddy will pay instant attention and maybe pick me up.

![](/assets/images/scna_extended/21.gif#scna_extended )

___

Relative sizes when the ratio is at least 2:1
* babies are small / parents are big

![](/assets/images/scna_extended/22.jpeg#scna_extended )

___

I wanted to share this video with you, when I watched it really blew my mind!

[to think](/assets/images/scna_extended/23.mp4 target="_blank)

[Full video link](https://youtu.be/ue1h1_vRLtU?t=22m21s)

If you look into the eyes of a 6 month old child, when he or she is in the park. She looks around and with that look she performs computation, she’s analyzing the world around her. How far mommy is, how she can get her attention, or grab her favorite toy

___

As you see in this graph, so much is happening in the first year of a baby’s life. There are roughly 2,000 days from the time a baby is born to when that child begins Kindergarten: Kids aren’t just on a five year vacation before they start in the K-12 school system!

Babies are born with 100 billion neurons, roughly about how many stars are in Milky Way and that’s about all the neurons you will ever have. But only about a quarter of the connections - synapses - are already made between them. The young brain makes billions of new connections with every bit of knowledge that is taken in, in order to make sense of it.

By age 2, the brain has up to twice as many synapses as it will have in adulthood. While the brain cells are formed before birth, the connections, the wiring that forms the architecture happens in infancy and early childhood. And how that wiring is formed, either as a strong or weak foundation depends on each child’s interactions with the world around him/her.

Of course that doesn’t mean as we age we’re not capable of learning new skills or enhancing them but it requires more effort. So much of we use now are the connections that we have built early in our lives and those that are not being used will get weaker and weaker.

Given the impact on brain development, it’s not surprising several studies emphasize the significant long-term impacts of creative environments.


![](/assets/images/scna_extended/24.jpeg#scna_extended )

___

In a great book, Mindstorms, Seymour Papert suggests we look at Mathematics as a second language.

Kids are pressured in K-12 to learn STEM after potentially having gone that 2000 days without mathematical or computational thinking. They can become afraid of math because of the unfamiliarity of material like symbolic notation as a model for a real-world pattern, the lack of immediate feedback, and a fear of failure of not being a “math person”.


![](/assets/images/scna_extended/25.jpeg#scna_extended )

___

Maryam Mirzakhani was a mathematician and professor at Stanford university, sadly passed away a few months go. She was well-known for being the first female who won the most prestigious award in mathematics, the Fields Medal, in 2014.

I love this quote from her:

> "The beauty of mathematics only shows itself to more patient followers."

When learning mathematics, or a complicated algorithm for that matter, you have to be patient and allow your brain to deeply understand the problem, and to enjoy the path to solve it.

![](/assets/images/scna_extended/26.jpeg#scna_extended )

___

### Where to start?

Now that we know math and computation are essential, where do we start? For kids or ourselves? Let’s not forget, it’s not as though every K-12 is already a programmer and computational thinker. They’re going to need to learn, too.

___

The first place that I started and personally love is sorting laundry! That’s not just education, that’s task delegation!

Now how to go from here:

![](/assets/images/scna_extended/28.jpeg#scna_extended )

___

to here:

Organize the closet by type of cloths, color, sleeve length and so on!

![](/assets/images/scna_extended/29.jpeg#scna_extended )

___

**How do we turn a basic daily chore into a computational thinking exercise?**

___

### Merge Sort!
Merge sort is a great solution!

Well here we go…
What if I call someone to come and do a merge sort live?

___

Kidding!

But seriously, how many people know this algorithm off the top of your head and can live code it like we’re in a bad job interview?

![](/assets/images/scna_extended/32.jpeg#scna_extended )

___

Better yet, how many of you who use a pre-implemented solution or can code it, could show us what merge sort looks like without a computer?

![](/assets/images/scna_extended/33.jpeg#scna_extended )

___

For those who are not familiar with the implementation

1.	Divide the unsorted list into n sublists, each containing 1 element (a list of 1 element is considered sorted).

2.	Repeatedly merge sublists to produce new sorted sublists until there is only 1 sublist remaining. This will be the sorted list.


![](/assets/images/scna_extended/34.gif#scna_extended )

___

### Merge Sort - Ruby Edition!

This is one example of merge sort in ruby

    def mergesort(array)
      return array if array.length == 1
      middle = array.length / 2
      merge mergesort(array[0...middle]), mergesort(array[middle..-1])
    end

    def merge(left, right)
      result = []

      until left.length == 0 || right.length == 0 do
        result << (left.first <= right.left ? left.shift : right.shift)
      end
    end


This is NOT how you want to introduce the algorithm to your 5 years old!    

![](/assets/images/scna_extended/confusedArmin.png#scna_extended )

We don't want to get hung up on teaching classes, Ruby, or any other syntax yet. Rather, we need to build some intuition in the child about different sorting algorithms, and how merge sort might work in particular.

Remember the first step in computation was, decomposition and understanding the problem. Last step is to instruct the computer to do it

___

In case of Armin, he likes cards and poker! Why not start with sorting there?

The first time I asked him to sort a stack of cards, he starts with an intuitive but computationally inefficient approach. Of course he looked for 1, then 2 and so on…

Then without saying much I started the merge sort and he became interested. I introduced it as a game that always produces a consistently correct result. Inevitably he started arguing that his approach is much better. But, what if we’re sorting stack of 20, 30, 40, 100 cards? And soon he realized he want to learn a better method.

He also likes to know that the work he does is valuable and appreciated. So he was more than happy to make this video and explain how he does the sorting!

[Armin Merge sort](/assets/images/scna_extended/36.mov" target="_blank)

Not only did he get an introduction to sorting, but he’s already seen his first divide and conquer algorithm. Divide and conquer is a family of algorithms and a useful pattern, and shows up in politics from time to time as well…something for another talk, perhaps.

___

Once he was comfortable with the algorithm and had done a fair amount of debugging, I introduced **iteration** and timing.

Patience, Persistence, and Repetition are very important character traits during practice. This is what gives us the tolerance for failed attempts, letting us learn from them instead of simply giving up. This is how we build deep learning, understanding, and mastery.

![](/assets/images/scna_extended/37.jpeg#scna_extended )

___

Is anybody here familiar with Set game?

Marsha Jean Falco invented this game in 1974 and introduced it to outside in 1990! There’s a really cool history behind it, I won’t go to the details but you’re welcome to read it.

The goal is to find three cards where each feature, when looked individually is either all the same OR all different.

Each card contains four features:
* color (red, purple or green)
* shape (oval, squiggle or diamond)
* number (one, two or three)
* shading (solid, striped or outlined)

When playing, the first person to see a SET, calls out and picks up the three cards that makes the SET
The person with most SETs wins

This is a speed game that encourages the players to apply the logic and speed in order to win.

![](/assets/images/scna_extended/38.jpeg#scna_extended )

___

It turns out that sets are also very important in mathematics.

The language of set theory can be used in the definitions of nearly all mathematical objects and is commonly employed as a foundational system for mathematics

Once you understand the set, then you can apply other computations such was union and intersection to it.

This is Keith Devlin, he is an author of 34 books and over 80 articles. Among them are Intro to mathematics and The Joy of Sets.

The Set game makes the concept of sets fun.

![](/assets/images/scna_extended/39.jpeg#scna_extended )

___

How to get from cards

to the Clojure definition of set?

When we eventually introduce the programming concept of a Set, like the Clojure sets here, Armin will have some familiar concepts to fall back on.

The shared information of the physical should ease the jump from the physical to the abstract, and vice versa.

![](/assets/images/scna_extended/40.jpeg#scna_extended )

___

Let’s build on this theme of connecting computation in the physical world with programmatic models in the computer.

Sometime earlier this year, I ran into this cute but purposeful toy on Kickstarter to teach kids to start thinking programmatically.

It sounded very interesting. At the time, I thought it would be just too early for Armin to start, since I hadn’t even started the techniques we just talked about. Eventually sometime around April or May, I ordered one. And it’s been great! Cubetto is inspired by Turtle math to translate real-world thinking to a programming language. And it’s been approved by the Montessori

Let’s see Armin’s interaction with Cubetto 101!

![](/assets/images/scna_extended/41.jpeg#scna_extended )

___

As you see, this robot doesn’t have a screen. But each step blinks on the board while Cubetto is performing it so the child can make the connection. When functions are used, as well as the step that the function is executing.

[Cubetto 101](/assets/images/scna_extended/42.mov" target="_blank)

___

Here is another great thing: Before Armin put in the commands, he pseudo-coded the algorithm he wanted Cubetto to execute on his notepad!

Then when we started putting the commands on the board, Armin realized it’s a pattern.
He used a function block to define the pattern, DRY up his “code”, and run the program!

![](/assets/images/scna_extended/43.jpeg#scna_extended )

___

We come back to Keith Devlin’s work: Mathematics is the science of patterns.
A big part of mathematics is describing some pattern you observe in the world in a way that is precise, repeatable, and provably correct. Sometimes those patterns are numeric, sometimes geometric, sometimes they’re patterns of human social networks.

At the end of the day, this is a big part of what we’re doing as programmers.

![](/assets/images/scna_extended/44.jpeg#scna_extended )

___

To translate the computational thinking from the physical world to the virtual, we come back to Papert’s work with Mindstorms.

He and his colleagues created the LOGO programming language and visualization environment in 1967.

![](/assets/images/scna_extended/45.jpeg#scna_extended )

___

For those of you who are interested, several languages have now implemented LOGO. This is the Clojure version of LOGO, called clojure-turtle.

For Armin, the next step, when he’s ready, is to project his patterns on the computer. Learner can then take a step farther and come up with some absolutely beautiful designs. Take a look:

![](/assets/images/scna_extended/46.jpeg#scna_extended )

___

![](/assets/images/scna_extended/47.jpeg#scna_extended )

___

![](/assets/images/scna_extended/48.jpeg#scna_extended )

___

For the pythonista’s in the audience, there’s python-turtle

![](/assets/images/scna_extended/49.jpeg#scna_extended )

And now we bounce back from the virtual to the physical: Before we introduce Armin to these kinds of drawings, we need to teach him about angles!

___

Recently Armin asked us to leave his bedroom door “cracked” when he goes to sleep.

Following our “Be precise!” rule, we started to ask him to what angle he wanted his door cracked. That made us make this Protractor floor math.

We have a door that actually opens 180 degrees, imagine how many times I got hit by that when we were trying our new floor-mat! But his bedroom door is closed at a 0 degree angle and fully open at 90 degrees.

There’s another interesting thing Armin discovered when he was learning to make this protractor.

He realized after we put the dots in 5 degree intervals, he needed to connect them with lines to produce a circle, see him tracing with orange marker and to the corner is where he tested his theory. move a little forward, turn a little to same direction

He started with a triangle, to square, pentagon, hexagon, octagon… and so on till he got a circle.

![](/assets/images/scna_extended/50.jpeg#scna_extended )

___

Now this is how our conversation goes most nights:

> "Armin: Con you please leave my door cracked?"

> "Mom/Dad: What angle?"

> "Armin: 90°"

> "Mommy: Hey, that's too much! How about 5°?"

> "Armin: Fine, I will meet you in the middle... 45°"

> **Unintended side-effect. He really does negotiate this way, completely one-sided**

> "Final: 10°"

___

In addition to spending 1:1 time with our kids, you can also take advantage of programs that are design to encourage STEM in early childhood education.

One of these organizations is Blue\|1647. The had an event in April and I was a volunteer at the event.

For those who are not familiar, Blue\|1647 is a technology innovation center and this even was designed to encourage children as young as 4 years old to approach problems in scientific way.

For example, design a parachute that safely lands an egg… just use your imagination how the place looked liked after that. Or try making a paper airplane that stays in the air longer and travels a longer distance… try, redesign, and try again

These are the techniques that are so crucial in computational thinking and when put in such fun way encourages the children to learn them without thinking they are learning STEM

![](/assets/images/scna_extended/52.jpeg#scna_extended )

___

When learning mathematics or learning to look at a problem computationally,

* Be very patient, be persistent and allow yourself or the learner to digest the information without being under the pressure
* Don’t set up high expectations, don’t take shortcuts to meet those expectations. Let things progress naturally

![](/assets/images/scna_extended/53.jpeg#scna_extended )

___

I know many of you want to go home tonight and start teaching or learning. Let me introduce you to Dr. Jeffery Karpicke and his great research and how to remember what you’ve learned.

Jeffery Karpicke is a 10 year faculty staff at Purdue University, in Cognitive Psychology department.

![](/assets/images/scna_extended/54.jpeg#scna_extended )

___

One of his research projects was about word-learning in 2011 — Karpicke & Bauernschmidt

He wanted to see how do we learn and persist information in long term memory. In this research, students were going to learn words like mashua (Mazoa) in Swahili.

They were divided in 4 groups. Students were shown the words, and it’s translation, then they had to type the translation. In one week they would be tested to see how much they remembered.

In trial, they were shown the words, they had to recall it and type the translation.

Learning:

* Group 1: studying without recalling
* Group 2: study, recall them till they had answered them right, then the word would drop out of the list
* Group 3: this group was practicing **massed retrieval** practice where the word was shows and was repeated 3 times immediately for retrieval
* Group 4: Repeat 3 times but with **spaced repetition technique**

Retrieval - One week after the learning session

[read the full article](http://www.apa.org/science/about/psa/2016/06/learning-memory.aspx)

So next time you are learning new information, keep Jeffrey Karpicke’s research in mind and give it try!

![](/assets/images/scna_extended/55.jpeg#scna_extended )

___

Here is another very important guideline:
When you are not wrong you don't come up with anything original!

We're pushing and expecting kids to be right and that's how we take originality and creativity away form them.

Learn from attempts, don’t fear them. Celebrate the effort…the pride from the wins will come naturally. Encourage them to keep trying and become resilient to things not working.

Revise your work, change your methodologies!

Look at the problem at different angles and persist till you succeed.

![](/assets/images/scna_extended/56.jpeg#scna_extended )

___

This is a Proud parent moment I wanted to share with you and the importance of looking at Math as a language!

By now, you know that this presentation was just an excuse for me to show Armin’s pictures because he’s so cute.. last one, I promise!

A couple of weeks ago, Armin came to my husband with this drawing from his class. He first filled the color palettes and then realized that there are mix of primary and secondary colors: He knows yellow and blue make green, for example.

So he decided to connect the secondary colors to the primary colors that make them.

He made an arrow going out of black, because he thought black is a combination of all colors!

This was amazing to us, because we’d never explicitly taught him this. But if you see Bobby’s talk, who’s next, you know he loves networks and the kids have seen him working on graphs many times.

Armin connected the colors and created a graph and he showed it to Bobby saying, “Look, Daddy…a graph!” Bobby was so proud and told him he’d made a color dependency graph.

![](/assets/images/scna_extended/57.jpeg#scna_extended )

___

Armin was so proud of himself, he went to his room and signed his work.

Mom and dad this picture is for both of you, this is my first graph

![](/assets/images/scna_extended/58.jpeg#scna_extended )

___

Now, I invite and encourage you to go home and try some of these techniques out for yourself. Look at the problems at a different angle, computationally! Here are a few links to the tools I’ve shown.

![](/assets/images/scna_extended/59.jpeg#scna_extended )

___

And as something special for SCNA:

![](/assets/images/scna_extended/60.jpeg#scna_extended )

___

Stay in touch, let me know how you are doing or if you have ideas you would like to share!

Tonight, I want you to implement merge sort with cards and have fun doing that!

![](/assets/images/scna_extended/61.jpeg#scna_extended )

___

![](/assets/images/scna_extended/62.jpeg#scna_extended )

___

![](/assets/images/scna_extended/63.jpeg#scna_extended )

___
