---
layout: post
title:  "Computational Thinking"
date:   2017-10-27 22:04:39
categories: computer science
---

<div class="container">
  <iframe src="https://player.vimeo.com/video/241197949" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
</div>

<div class="container">
  <p>
    When I was first asked to speaking at SCNA, I know exactly what I wanted to talk about, Computational Thinking and how early we can start. It was almost a year since we
    started working with my older son, Armin to teach him to think computationally and ask questions. As time passed and I was preparing for my talk,
    I started having doubts about whether or not this topic will be of interest to others, specially those without kids. I had these doubts until about an hour before my talk!
    During the talk, I was looking at the audience, seeing them engaged, nodding and laughing. That, of course, made me happy. I felt like I was able to connect with them.
    But after my talk, during the Q&A, I was pleasantly surprised and happy with so many people come to me and asked variety of questions from their own experience,
    learning in general, or teaching their children. One professor even came to me and talked about how he is trying to make education and programing fun for his students.
    One student mentioned that's how he felt about learning programing language but hadn't really seen anybody talk about it before. Another student said, he was introduced to
    programing syntax first before learning to think computationally, and he's been struggling with learning how to program.
  </p>
  <p>
    It was very heartwarming to see I was able to connect to so many people at SCNA. Therefore, I decided to put the content in my blog and make it available to everyone.
    Hope this helps.
  </p>
</div>

<section>
  <title>Computational Thinking for kids of all ages!</title>

  <div class="container">
    <p>
      I've been an engineer for the past 14 years, but probably have been only thinking computationally for the last 7. To explain what I mean by that, let's start with talking about
      what computational thinking is NOT.
    </p>

    <p>
      Computational thinking is NOT thinking in programming language syntax. I think the worst thing we could do to get someone interested in programming is to introduce them to
      syntaxes like Java, C, or Fortran on day one. Why would they want to bother learning these unfamiliar, arcane details like pointers, classes, and compilation? Rather,
      let's start with a goal. What problem are they trying to solve? Can this problem be framed as a computational one that a computer can do much faster than we could?
    </p>

    {% highlight c++ %}
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
    {% endhighlight %}
  </div>

  <div class="container">
    <div class="left text">
      Nowadays, there are over 256 programing languages. The first program I wrote was over 20 years ago in DOS. When I started programming professionally, 14 years ago, I always gave it my all to deliver the project. But it took me some time to really go deep to try to understand <strong>why</strong> the program was going to work before I even wrote it.
    </div>
    <div class="right image">
      <img class="programing-language-icon" src="/assets/images/scna_extended/java.png" />
      <img class="programing-language-icon" src="/assets/images/scna_extended/clojure.png" />
      <img class="programing-language-icon" src="/assets/images/scna_extended/dotNet.png" />
      <br />
      <img class="programing-language-icon" src="/assets/images/scna_extended/elixir.png" />
      <img class="programing-language-icon" src="/assets/images/scna_extended/react.png" />
      <img class="programing-language-icon" src="/assets/images/scna_extended/c++.png" />
    </div>
  </div>

  <div class="container">
    <div class="left text">
      In 2010, I left ThoughtWorks and started my own consultancy, Tested Minds. Which then allowed me to really think and pick projects that I was really interested in. Then, I made my first project from a genuine desire to have a computer solve a problem for me. I wanted to teach my husband read and write Farsi. I wanted him to practice as much as he wanted too. But I didn’t have time or was available all the time to sit with him.
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/04.jpeg#scna_extended" />
    </div>
  </div>

  <div class="container">
    <p>
      The computation I wanted literately to do included things like:
    </p>
    <div class="row">
      <div class="left image">
        <img class="image-small" src="/assets/images/scna_extended/literatelyExample1.png" />
        <img class="image-small" src="/assets/images/scna_extended/literatelyExample2.png" />
      </div>
      <div class="right text">
        Provide a user with immediate feedback about whether they'd recognized the character.
      </div>
    </div>
    <div class="row">
      <div class="left image">
        <img class="image-small" src="/assets/images/scna_extended/literatelyExample3.png" />
      </div>
      <div class="right text">
        Show aggregated stats about a user's progress.
      </div>
    </div>

    <p>
      My mother is a retired elementary teacher, and most of her career she taught first grade. She had performed these computations for a long time! Her and I put a curriculum together which I then I used Ruby on Rails to build the application and put it online for free as literately-Farsi!
    </p>
  </div>

  <div class="container">
    <div class="left text">
      <p>
        There’s this famous quote:
        <blockquote>
          "Computer science is no more about computers than astronomy is about telescopes."
        </blockquote>
      </p>
      <p>
        Sure, I agree with it to some level, but learning about astronomy without <strong>ever</strong> using a telescope, I'd wager, would be really frigging hard.
        Learning to think computationally without ever writing code or using computers, similarly, holds back a learner.
        I think the question is really about <strong>when</strong> to introduce code, not if.
      </p>
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/edsgerWybeDijkstra.jpg" />
    </div>
  </div>

  <div class="container">
    <p>
      So this raises two questions:
      <ul>
        <li>How early can we introduce computational thinking?</li>
        <li>How early can we introduce programming?</li>
      </ul>
    </p>
  </div>
</section>

<section>
  <title>Let’s look at computation vs. arithmetic</title>
  <div class="container">
    <div class="left text">
      The word "computation" is often used synonymously to mean "numeric computation". Computation is the act of processing the data, calculation and reckoning
      Arithmetic is type of mathematics, integers, real numbers and complex numbers. We can apply addition, subtraction, multiplication, and division to those numbers
      Mathematics is the way to precisely represent data and proof a formula. In order to do computation and for the computers to be able to run algorithm we need to be quite precise, that’s why mathematics is important in computation
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/07.jpeg" />
    </div>
  </div>
</section>

<section>
  <title>Human brain vs. computers</title>
  <div class="container">
    <div class="left text">
      <p>
        Let’s look at the Fibonacci Sequence example n = (n-1) + (n-2):
      </p>
      <p>
        Without using any “computational devices”, most of us will slow down around the 10th digit. Let’s see this in action, try this along with me.
        <br />
        Given 0 (n-2) and 1 (n-1), what comes next in the sequence?
      </p>
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/fib.jpg" />
    </div>
  </div>
  .
  <br />
  .
  <br />
  .
  <br />

  <div class="container">
    <p>
      How many numbers we were able to calculate before you slow down?
      <br />
      Of course some of you geniuses can go easily up to 15…20…30…digits. But you’re weird…in a good way! :)
    </p>

    <p>
      How about this one?
      <br />
      <strong>What's the 100th number in the Fibonacci sequence?</strong>
      <br />
      How long would it take a human brain to get this answer? And how likely would we be to make a mistake along the way because of a bug in our brain’s programming?
    </p>

    <p>
      This is where a computer can really help us out.
      <ul>
        <li>How long does it take for us to write that in our language of choice <i>once we know an algorithm to generate the sequence</i>?</li>
        <li>How long would it take for a computer to run that code?</li>
      </ul>
    </p>

    <p>
     Clojure syntax:

     {% highlight clojure %}
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
     {% endhighlight %}
    </p>

    <p>
      Now it's find out the 100th number:

      {% highlight clojure %}
      (nth fib 100)
      {% endhighlight %}

      <strong>345,224,848,179,261,915,075</strong>  - Elapsed time: 0.059953 msecs
      <br />
      Now not all implementations of this algorithm will be this performant!
    </p>

    <p>
      <strong>Computer science</strong>, especially programming language theory and implementation is what lets us have a conversation about things like lazy sequences, tail-call optimization, and stack frames.
    </p>
    <p>
      <strong>Computational thinking</strong>, on the other hand, is what lets us design an algorithm that can produce the Fibonacci sequence in the first place…with or without a computer.
    </p>
    <p>
      Before we write the code, we need to frame the problem in a precise, repeatable recipe, <strong>an algorithm</strong>. What needs to happen in between the inputs we have and the output we want? How will these inputs and outputs be modeled? What data structures will we need for that modeling?
    </p>
    <p>
      I’ll make the claim that there’s nothing the computer can do that the human brain cannot, given enough time and memory.
    </p>
  </div>

  <div class="container">
    <h4>Are the computers always faster and more efficient?</h4>
    <div class="left text">
      Remember captcha? We all love it so much!
      <br/>
      Noisy image recognition is a computational problem that humans can do subconsciously, but that computers struggle with. We could have a whole talk about the work going on with artificial neural networks to solve this type of problem, but what if we just put a human brain to work, for now?
      <br />
      And that’s what Luis Von Ahn did!
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/13.jpeg" />
    </div>
  </div>

  <div class="container">
    <div class="left text">
      <a href="https://www.mturk.com/mturk/welcome" target="blank">How about Amazon Mechanical Turk</a>
      <br />
      This is an example of what our brain can read and comprehend with one look but computers are not yet sophisticated enough to do. Most humans can't do numeric manipulation quickly. Most computers can't do free-form pattern-matching the way our brains can.
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/14.jpeg" />
    </div>
  </div>
</section>

<section>
  <title>Human Computing</title>
  <div class="container">
    <div class="row">
      <div class="left text">
        <p>
          This is Grady Booch, he’s a software engineer and some of us might know him as one of the guys behind UML… but he’s spending much of his time these days in computation and AI.
        </p>
        <p>
          I love this quote from him:
          <blockquote>
            Story of computing is the story of humanity!
          </blockquote>
        </p>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/gradyBooch.jpg" />
      </div>
    </div>

    <div class="row">
      <div class="left text">
        <p>
          Computing started out as human computers, mostly women. Computation, preformed mostly by women, in WWII was one of the key factors we won the war!
        </p>
        <p>
          In one of his articles, Grady mentions:
          <blockquote>
            Computational thinking drove our imagination, it goes beyond what computers can do and what we can do in present time
          </blockquote>
        </p>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/16.jpeg" />
      </div>
    </div>

    <div class="row">
      <div class="left image">
        <img src="/assets/images/scna_extended/17.jpeg" />
        <br />
        <strong>We are still the most extraordinary computers of all.</strong>
      </div>
      <div class="right text">
        We are computing on a daily basis. We planned our route to this conference. We performed a scheduling algorithm to arrive to a place on time. We just might not have done any of this consciously.
        <br />
        How do we gain skill in <strong>consciously</strong> thinking of problems computationally? And if we can do so as adults, how early can humans start? Can we make computational and mathematical thinking like a second language at home to teach our kids while we practice ourselves?
      </div>
    </div>
  </div>
</section>

<section>
  <title>How early we can start?</title>

  <div class="container">
    <div class="left text">
      Well, as a mother of two, I have a couple of research subjects that volunteer for all of my experiments!

      <ul>
        <li>How do I prepare my children to be excited about problem solving in their world?</li>
        <li>How do I teach them to look at numbers and analyze the facts, not just listen to other people’s opinions?</li>
        <li>How I prepare them to take a complex problem in life and break it down till they find a solution?</li>
      </ul>
      My bet is that computational thinking is a great way to start.
    </div>
    <div class="right image">
      <figure>
        <img src="/assets/images/scna_extended/18.jpeg" />
        <figcaption>
          Aidin 4 and Armin 6 years old.
        </figcaption>
      </figure>
    </div>
  </div>

  <div class="container">
    <div class="left image">
      <img src="/assets/images/scna_extended/19.jpeg" />
    </div>
    <div class="right text">
      <p>
        Now let’s talk research ethics here for a second. Before you accuse me of being a mad scientist and treating my kids like minions, let me show you some of the homework I have done.
      </p>
      <p>
        When we were expecting our first child six years ago, we were complete apprentices in parenthood. So my husband and I started reading a lot of books and watching a lot of documentaries about early childhood education and infant brain development.
      </p>
    </div>
  </div>

  <div class="container">
    <p>
      At just a few months old, babies can:
    </p>

    <div class="row">
      <div class="left text">
        <strong>predict sequences of events:</strong>
        <ul>
          <li>running water => bath time</li>
          <li>mommy goes to fridge with a glass => water</li>
          <li>putting shoes on => go outside</li>
        </ul>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/fridge.jpg" />
      </div>
    </div>

    <div class="row">
      <div class="left text">
        <strong>Connect cause and effect:</strong>
        <ul>
          <li>shaking a rattle => makes a sound</li>
          <li>pushing a toy off the table => drop, bang…the joy of gravity!</li>
          <li>cry => Mommy or Daddy will pay instant attention and maybe pick me up.</li>
        </ul>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/21.gif" />
      </div>
    </div>

    <div class="row">
      <div class="left text">
        <strong>Understand relative sizes when the ratio is at least 2:1</strong>
        <ul>
          <li>babies are small / parents are big</li>
          <li>line 1 is shorter than line 2</li>
        </ul>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/likeFatherLikeSon.jpg" />
      </div>
    </div>
  </div>

  <div class="container">
    <p>
      Babies are born with innate capacity for understanding numbers. This is an amazing video done by psychologists at Karen Waynn's Lab at Yale University.
    </p>
    <p>
      In this segment, the researchers show a doll to a six-month old in a setup similar to a puppet show. Then a curtain covers the stage where
      the doll is sitting, and the child could see another puppet go behind the curtain. Then the researcher sneakily removes the first puppet
      without the baby being able to see it. The curtain falls and only one puppet remains.
      <br />
      <strong>1 + 1 = 1?</strong>
    </p>
    <p>
      The baby is astonished and starts staring and banging the table! Every time the curtain was lifted and the answer was what they expected,
      however, the baby looked bored and looked away after only a few seconds. Dr. Wynn and her team replicated these results with six-month olds
      and even four-month olds, demonstrating that babies recognize addition and subtraction if only subconsciously.
      <br />
      Think about that! A baby can recognize mathematical inconsistency in the real world years before they can recognize a grammatical model of
      the world like “2 – 1 = 1”!
    </p>

    <div class="container">
      <video controls="controls" name="To Think" src="/assets/images/scna_extended/23.mp4"></video>
    </div>

    <p>
      If you look into the eyes of a 6-month-old child, when he or she is in the park, you will notice that she is performing computation and analyzing the world around her.
      How far mommy is, how she can get her attention, or grab her favorite toy.
    </p>
  </div>

  <div class="container">
    <div class="left text">
      <p>
        As we see in this graph, so much is happening in the first year of a baby’s life. There are roughly 2,000 days from the time a baby is born to when that child begins Kindergarten.
        Kids aren’t just on a five-year vacation before they start in the K-12 school system!
      </p>
      <p>
        Babies are born with 100 billion neurons, roughly about how many stars are in Milky Way and that’s about all the neurons we will ever have.
        But only about a quarter of the connections - synapses - are already made between them. The young brain makes billions of new connections with every bit of knowledge that
        is taken in, in order to make sense of it.
      </p>
      <p>
        By age 2, the brain has up to twice as many synapses as it will have in adulthood. While the brain cells are formed before birth, the connections,
        the wiring that forms the architecture happens in infancy and early childhood. And how that wiring is formed, either as a strong or weak foundation depends on each child’s
        interactions with the world around him/her.
      </p>
      <p>
        Of course that doesn’t mean as we age we’re not capable of learning new skills or enhancing them but it requires more effort. So much of we use now are the connections
        that we have built early in our lives and those that are not being used will get weaker and weaker.
      </p>
      <p>
        Given the impact on brain development, it’s not surprising several studies emphasize the significant long-term impacts of creative environments.
      </p>
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/24.jpeg" />
    </div>
  </div>

  <div class="container">
    <div class="left text">
      <p>
        In a great book, Mindstorms, Seymour Papert suggests we look at Mathematics as a second language.
      </p>
      <p>
        Kids are pressured in K-12 to learn STEM after potentially having gone those 2000 days without mathematical or computational thinking.
        They can become afraid of math because of the unfamiliarity of material like symbolic notation as a model for a real-world pattern,
        the lack of immediate feedback, and a fear of failure of not being a “math person”.
      </p>
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/seymour.jpg" />
      <br />
      <img src="/assets/images/scna_extended/mindstorms.jpg" />
    </div>
  </div>

  <div class="container">
    <h4>Be Patient when Learning!</h4>
    <p>
      Dr. Maryam Mirzakhani was a mathematician and professor at Stanford University, sadly passed away a few months go. She was well known
      for being the first female who won the most prestigious award in mathematics, the Fields Medal, in 2014.
    </p>
    <div class="row">
      <div class="left text">
        I love this quote from her:
        <br />
        <blockquote>
          The beauty of mathematics only shows itself to more patient followers.
        </blockquote>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/maryamMirzakhani.jpg" />
      </div>
    </div>
    <p>
      When learning mathematics, or any complicated algorithm for that matter, we have to be patient and allow our brain to deeply understand the problem, and to enjoy the path to solve it.
    </p>
  </div>
</section>

<section>
  <title>Where to start?</title>
  <div class="container">
    <p>
      Now that we know math and computation are essential, where do we start? For kids or ourselves? Let’s not forget, it’s not as though every K-12 is already a programmer and computational thinker, they’re going to need to learn, too.
    </p>
    <p>
      The first place that I started and personally love is sorting laundry! That’s not just education, that’s task delegation!

      <strong>How do we turn a basic daily chore into a computational thinking exercise?</strong>

    </p>
    <div class="row">
      <div class="left text">
        Now how to go from here:
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/28.jpeg" />
      </div>
    </div>

    <div class="row">
      <div class="left text">
        To here:
        <br />
        Organize the closet by type of cloths, color, sleeve length and so on!
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/29.jpeg" />
      </div>
    </div>
  </div>


  <h4>Merge Sort!</h4>

  <div class="container">
    <p>
      Merge sort is a great solution!
    </p>
    <p>
      How many people know this algorithm off the top of your head and can live code it like we’re in a bad job interview?
    </p>
    <p>
      Better yet, how many of us who use a pre-implemented solution or can code it, could show us what merge sort looks like without a computer?
    </p>
  </div>

  <div class="container">
    <div class="left text">
      For those who are not familiar with the implementation (from wiki):
      <ol>
        <li>Divide the unsorted list into n sublists, each containing 1 element (a list of 1 element is considered sorted)</li>
        <li>Repeatedly merge sublists to produce new sorted sublists until there is only 1 sublist remaining. This will be the sorted list.</li>
      </ol>
    </div>
    <div class="left image">
      <img src="/assets/images/scna_extended/34.gif" />
    </div>
  </div>

  <div class="container">
    <h4>Merge Sort - Ruby Edition!</h4>

    <p>
      This is one example of merge sort in ruby:

      {% highlight ruby %}
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
      {% endhighlight %}
    </p>
    <div class="row">
      <div class="left image">
        <img class="armin_confused_picture" src="/assets/images/scna_extended/confusedArmin.png" />
      </div>
      <div class="right text">
        This is NOT how we want to introduce the algorithm to a 5-year old!
      </div>
    </div>
    <p>
      We don't want to get hung up on teaching classes in Ruby or any other syntax yet. Rather, we need to build some intuition in the child about different sorting algorithms and how merge sort might work in particular.
    </p>
    <p>
      Remember the first step in computation was, decomposition and understanding the problem. Last step is to instruct the computer to do it
    </p>
  </div>

  <div class="container">
    <p>
      In case of Armin, he likes cards and poker! Why not start with sorting there?
    </p>

    <p>
      The first time I asked him to sort a stack of cards, he starts with an intuitive but computationally inefficient approach. Of course he looked for 1, then 2 and so on…
      <br />
      Then without saying much I started the merge sort and he became interested. I introduced it as a game that always produces a consistently correct result. Inevitably, he started arguing that his approach is much better. But, what if we’re sorting stack of 20, 30, 40, 100 cards? And soon he realized he want to learn a better method.
    </p>
    <p>
      He also likes to know that the work he does is valuable and appreciated. So he was more than happy to make this video and explain how he does the sorting!
    </p>
    <div class="container">
      <video controls="controls" name="Merge Sort" src="/assets/images/scna_extended/36.mov" />
    </div>
    <p>
      Not only did he get an introduction to sorting, but he’s already seen his first divide and conquer algorithm. Divide and conquer is a family of algorithms and a useful pattern, and shows up in politics from time to time as well…something for another talk, perhaps.
    </p>
  </div>

  <div class="container">
    <div class="left text">
      <p>
        Once he was comfortable with the algorithm and had done a fair amount of debugging, I introduced <strong>iteration</strong> and timing.
      </p>
      <p>
        Keep in mind that patience, persistence, and repetition are very important character traits during practice. This is what gives us the tolerance for failed attempts, letting us learn from them instead of simply giving up. This is how we build deep learning, understanding, and mastery.
      </p>
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/37.jpeg" />
    </div>
  </div>

  <h4>Set</h4>

  <div class="container">
    <div class="row">
      <div class="left text">
        <p>
          Marsha Jean Falco invented this game in 1974 and introduced it to outside in 1990! There’s a really cool history behind it; I won’t go to the details but you’re welcome to read it.
        </p>
        <p>
          The goal is to find three cards where each feature, when looked individually is either all the same OR all different.
        </p>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/setGame.png" />
      </div>
    </div>
    <p>
      Each card contains four features:
      <ul>
        <li>color (red, purple or green)</li>
        <li>shape (oval, squiggle or diamond)</li>
        <li>number (one, two or three)</li>
        <li>shading (solid, striped or outlined)</li>
      </ul>
    </p>
    <p>
      When playing, the first person to see a SET, calls out and picks up the three cards that makes the SET. The person with most SETs (or cards)
      wins.
      <br />
      This is a speed game that encourages the players to apply the logic and speed in order to win.
    </p>
  </div>

  <div class="container">
    <p>
      It turns out that sets are also very important in mathematics. The language of set theory can be used in the definitions of nearly all 
      mathematical objects and is commonly employed as a foundational system for mathematics. Once we understand the set, then we can apply
      other computations such was union and intersection to it.
    </p>
    <div class="row">
      <div class="left text">
        This is Keith Devlin, he is an author of 34 books and over 80 articles. Among them are Intro to mathematics and The Joy of Sets.
        <blockquote>
          <h5>
            <strong>The Joy of Sets</strong>
          </h5>
          In naive set theory we assume the existence of some given domain of “objects”, out of which we may build sets.
          Just what these objects are is of no interest to us. Our only concern is the behavior of the “set” concept.
        </blockquote>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/keithDevlin.jpg" />
      </div>
    </div>
    <p>
      The Set game makes the concept of sets fun.
    </p>
  </div>

  <div class="container">
    <div class="row">
      <div class="left text">
        How to get from cards:
      </div>
      <div class="left image">
        <img src="/assets/images/scna_extended/set.png" />
      </div>
    </div>
    <div class="row">
     To the Clojure definition of set?
     {% highlight clojure %}
     #{:red :purple :green}
     #{:oval :squiggle :diamond}
     #{1 2 3}
     #{:solid :striped :outlined}
     {% endhighlight %}
   </div>
   <p>
    When we eventually introduce the programming concept of a Set, like the Clojure sets here, Armin will have some familiar concepts to fall back on.
    </p>
    <p>
      The shared information of the physical should ease the jump from the physical to the abstract, and vice versa.
    </p>
    <p>
      <blockquote>
        Theory: The information learned from computational thinking about the familiar physical world makes it easier to model that information
        in a programming language later on in a potentially <u>unfamiliar virtual world…</u> and vice versa.
      </blockquote>
    </p>
  </div>

  <h4>Cubetto</h4>

  <div class="container">
    <div class="left text">
      <p>
        Let’s build on this theme of connecting computation in the physical world with programmatic models in the computer.
      </p>
      <p>
        Sometime earlier this year, I ran into this cute but purposeful toy on Kickstarter to teach kids to start thinking programmatically.
        It sounded very interesting. At the time, I thought it would be just too early for Armin to start, since I hadn’t even started the techniques
        we just talked about. Eventually sometime around April or May, I ordered one and it’s been great! Cubetto is inspired by Turtle math to
        translate real-world thinking to a programming language and it’s been approved by the Montessori
      </p>
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/41.jpeg" />
    </div>
  </div>

  <div class="container">
    <p>
      Let’s see Armin’s interaction with Cubetto 101! As we see, this robot doesn’t have a screen. However, each step blinks on the board when
      functions are used as well as the step that's being executed, while Cubetto is performing it so the child can make the connection.
    </p>
    <div class="container">
      <video controls="controls" name="Cubetto 101" src="/assets/images/scna_extended/42.mov" />
    </div>
    <div class="row">
      <div class="left text">
        <p>
          Here is another great thing: Before Armin put in the commands, he pseudo-coded the algorithm he wanted Cubetto to execute on his notepad!
        </p>
        <p>
          Then when we started putting the commands on the board, Armin realized it’s a pattern. He used a function block to define the pattern,
          DRY up his “code”, and run the program!
          <ul>
            <li>Visually seeing bugs and fix them</li>
            <li>Algorithm development</li>
            <li>Functions</li>
          </ul>
        </p>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/arminCubettoSudoCode.jpg" />
      </div>
    </div>
  </div>

  <div class="container">
    <div class="left text">
      <p>
        Here we come back to Keith Devlin’s work: Mathematics is the science of patterns. A big part of mathematics is describing some pattern we
        observe in the world in a way that is precise, repeatable, and provably correct. Sometimes those patterns are numeric, sometimes geometric
        and sometimes they’re patterns of human social networks.
      <br />
        At the end of the day, this is a big part of what we’re doing as programmers.
      </p>
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/scienceOfPatternsBook.jpg" />
    </div>
  </div>

  <h4>LOGO</h4>

  <div class="container">
    <div class="left text">
      To translate the computational thinking from the physical world to the virtual, we come back to Papert’s work with Mindstorms.
      He and his colleagues created the LOGO programming language and visualization environment in 1967.
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/logo.png" />
    </div>
  </div>

  <div class="container">
    <div class="row">
      For those of you who are interested, several languages have now implemented LOGO. This is the Clojure version of LOGO, called clojure-turtle.
      <div class="left text">
        {% highlight clojure %}
        (repeat 4 (all (forward 30) (right 90)))
        {% endhighlight %}
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/turtleClojure.png" />
      </div>
    </div>
    <p>
      For Armin, the next step, when he’s ready, is to project his patterns on the computer.
    </p>

    <p>
      Learner can then take a step farther and come up with some absolutely beautiful designs. Take a look:
    </p>

    <p>
      <img class="image-medium" src="/assets/images/scna_extended/turtleTrees.png" />
    </p>
    <p>
      <img class="image-medium" src="/assets/images/scna_extended/turtleFun.png" />
    </p>
    <p>
      For the pythonista’s in the audience, there’s python-turtle
    <p>
    </p>
      <img class="image-medium" src="/assets/images/scna_extended/turtleSpiral.png" />
    </p>
  </div>

  <h4>Degrees</h4>

  <div class="container">
    <p>
      Now we bounce back from the virtual to the physical. Before we introduce Armin to these kinds of drawings,
       we need to teach him about angles!
    </p>
    <div class="row">
      <div class="left text">
        Recently Armin asked us to leave his bedroom door “cracked” when he goes to sleep. Following our “Be Precise!” rule, we started to ask
        him to what angle he wanted his door cracked. That made us make this Protractor floor math. We have a door that actually opens 180 degrees,
        imagine how many times I got hit by that when we were trying our new floor-mat! But, luckily, his bedroom door is closed at a 0 degree angle
        and fully open at 90 degrees!
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/arminProtractor1.jpg" />
      </div>
    </div>
    <div class="row">
      <div class="left text">
        There’s another interesting thing Armin discovered when he was learning to make this protractor. He realized after we put the dots in
        5 degree intervals, he needed to connect them with just straight lines to produce a circle, see him tracing with orange marker and to
        the corner is where he tested his theory. Move a little forward, turn a little to same direction.
        He started with a triangle, to square, pentagon, hexagon, octagon… and so on till he got a circle.
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/arminProtractor2.jpg" />
      </div>
    </div>

    <p>
      Now this is how our conversation goes most nights:

      <blockquote>
        Armin: Con you please leave my door cracked?
        <br />
        <br />
        Mom/Dad: What angle?
        <br />
        <br />
        Armin: 90°
        <br />
        <br />
        Mommy: Hey, that's too much! How about 5°?
        <br />
        <br />
        Armin: Fine, I will meet you in the middle... 45°
        <br />
        <br />
        <strong>Unintended side-effect. He really does negotiate this way, completely one-sided</strong>
        <br />
        <br />
        Final: 10°
    </blockquote>
    </p>
  </div>

  <h4>STEM Workshops</h4>
  <div class="container">
    <div class="left text">
      <p>
        In addition to spending 1:1 time with our kids, we can also take advantage of programs that are design to encourage STEM in early
        childhood education. One of these organizations is Blue\|1647.
      </p>
      <p>
        For those who are not familiar, Blue\|1647 is a technology innovation center and this even was designed to encourage children as young
        as 4 years old to approach problems in scientific way.
      </p>
      <p>
        Recently, they had an event in April that I volunteered for. I worked with a group of 8-10 years old on a day full of STEM workshops.
        For example, we design a parachute that safely lands an egg… just use your imagination how the place looked liked after that.
        Or try making a paper airplane that stays in the air longer and travels a longer distance… test, redesign/debug, and test again.
      </p>
      <p>
        These are the techniques that are so crucial in computational thinking and when put in such fun way, it encourages the children to
        learn them without thinking they are learning STEM.
      </p>
    </div>
    <div class="right image">
      <img src="/assets/images/scna_extended/blue1.jpg" />
      <br />
      <img src="/assets/images/scna_extended/blue2.jpg" />
    </div>
  </div>
</section>

<section>
  <title>Guidelines</title>
  <div class="container">
    <p>
      When learning mathematics or learning to look at a problem computationally. Be very patient, be persistent and allow ourselves or the
      learner to digest the information without being under the pressure. Don’t set up high expectations, don’t take shortcuts to meet
      those expectations. Let things progress naturally
    </p>
    <ul>
      <li>Be patient</li>
      <li>Be persistent</li>
      <li>Do not set high expectations</li>
      <li>Do not teach programing syntax yet</li>
      <li>Make it about day-to-day life, be precise
        <ul>
          <li>what temperature should the oven be on?</li>
          <li>how many more minutes you need to play?</li>
        </ul>
      </li>
      <li>Let the child manage money</li>
      <li>Show them fractals</li>
      <li>Have them watch our speed when driving (warning: this could be problematic!)</li>
      <li>Don't fear mistakes!
        <ul>
          <li>When we are not wrong, we don't come up with anything original! We are pushing and expecting our kids to be always right; and that's how we take originality and creativity away from them.</li>
          <li>Learn from attempts, don't fear them. Celebrate the effort, the pride form the wins will come naturally. Encourage children to keep trying and become resilient to things not working.</li>
          <li>Revise our work, change our methodologies!</li>
          <li>Don't tell our children they are smart, even if the tests say are! It dismisses the hard work they are doing, and have to continue doing, to achieve their goals.</li>
          <li>Let our children get bored from time to time to encourage their creativity.</li>
        </ul>
      </li>
    </ul>
  </div>
</section>

<section>
  <title>Spaced Repetition Learning</title>
  <div class="container">
    <div class="row">
      <div class="left text">
        I know many of us want to go home tonight and start teaching or learning. Let me introduce you to Dr. Jeffery Karpicke and his great research and how to remember what you’ve learned.
        <br />
        Jeffery Karpicke is a 10-year faculty staff at Purdue University, in Cognitive Psychology department.
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/jeffreyKarpicke.png" />
      </div>
    </div>
    <div class="row">
      <div class="left text">
        His areas of interest include:
        <ul>
          <li>Human Learning and memory, especially retrieval process</li>
          <li>Cognitive Science and education</li>
          <li>Complex Learning, comprehension, and knowledge application</li>
          <li>Learning and cognitive strategies in children</li>
          <li>Metacognition and self-regulated learning</li>
          <li>Education technology and computer-based learning</li>
        </ul>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/jeffreyKarpickeHIdex.png" />
      </div>
    </div>
  </div>

  <div class="container">
    <p>
      Karpicke and Bauernschmidt did a research projects on about word-learning in 2011. They wanted to see how do we learn and persist
      information in long term memory. In this research, students were going to learn words like "mashua" in Swahili.
    </p>
    <p>
      Students were divided into 4 groups. They were shown the words, and it’s translation, then they had to type the translation (recall).
      In one week they would be tested to see how much they remembered.
    </p>
    <div class="row">
      <div class="left text">
        <p>
          Learning:
          <ul>
            <li>Group 1: studying without recalling</li>
            <li>Group 2: study, recall them till they had answered them right, then the word would drop out of the list</li>
            <li>Group 3: this group was practicing <strong>massed retrieval</strong> practice where the word was shows and was repeated 3 times immediately for retrieval</li>
            <li>Group 4: Repeat 3 times but with <strong>spaced repetition technique</strong></li>
          </ul>
        </p>
      </div>
      <div class="right image">
        <figure>
          <img src="/assets/images/scna_extended/spaceRepititionLearning.png" />
          <figcaption>
            Retrieval: One week after the learning session - <a href="http://www.apa.org/science/about/psa/2016/06/learning-memory.aspx" target="blank">Read the full article</a>.
          </figcaption>
        </figure>
        </div>
      </div>
    <p>
      <strong>Repeated retrieval with immediate feedback leads to long term retention- Jeffrey Karpicke</strong>
    </p>
    <p>
      So next time we are learning new information, let's keep Jeffrey Karpicke’s research in mind and give it try!
    </p>
  </div>
</section>

<section>
  <title>Math Language</title>
  <div class="container">
    <p>
      This is a proud parent moment I wanted to share with you and the importance of looking at Math as a language!
    </p>
    <p>
      By now, you know that this blog was just an excuse for me to show Armin’s pictures because he’s so cute... last one, I promise!
    </p>
    <div class="row">
      <div class="left text">
        <p>
          A couple of weeks ago, Armin came to my husband with this drawing from his class. He first filled the color palettes and then realized
          that there are mix of primary and secondary colors, He knows yellow and blue make green, for example. So he decided to connect the secondary
          colors to the primary colors that make them. He made an arrow going out of black, because he thought black is a combination of all colors!
          This was amazing to us, because we’d never explicitly taught him this. But for those who know Bobby and have attended his talks, you know he
          loves networks and the kids have seen him working on graphs many times. Armin connected the colors and created a graph and he showed it to
          Bobby saying, “Look, Daddy… a graph!” Bobby was so proud and told him he’d made a color dependency graph.
        </p>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/arminGraph.jpg" />
      </div>
    </div>
    <div class="row">
      <div class="left text">
        Armin was so proud of himself; he went to his room and signed his work.
        <blockquote>
          Mom and dad this picture is for both of you, this is my first graph!
        </blockquote>
      </div>
      <div class="right image">
        <img src="/assets/images/scna_extended/arminGraphNote.png" />
      </div>
    </div>
  </div>
</section>

<section>
  <title>Toys and Games in this Blog</title>
  <div class="container">
    <ul>
      <li><a href="https://www.primotoys.com/" target="blank">Cubetto</a></li>
      <li><a href="http://amzn.to/2iI0ttA" target="blank">SET</a></li>
      <li><a href="https://github.com/google/clojure-turtle" target="blank">Clojure Turtle</a></li>
    </ul>
  </div>
</section>

<section>
  <div class="container">
    And as something special for SCNA:
    <img src="/assets/images/scna_extended/61.jpeg" />
  </div>
</section>

<section>
  <div class="container">
    Stay in touch; let me know how you are doing or if you have ideas you would like to share!
    <ul>
      <li><a href="http://www.linkedin.com/in/felfeli" target="blank">linkedin/felfli</a></li>
      <li>sarah@testedminds.com</li>
    </ul>
  </div>
</section>

<section>
  <title>References</title>
  <div class="container">
    <li><a href="http://www.apa.org/science/about/psa/2016/06/learning-memory.aspx" target="blank">Jeffrey Karpicke - Learning Memory</a></li>
    <li><a href="http://amzn.to/2gRrzy2" target="blank">Keith Devlin - Introduction to Mathematics</a></li>
    <li><a href="http://amzn.to/2xvRfU3t" target="blank">Keith Devlin - The Joy of Sets</a></li>
    <li><a href="http://amzn.to/2zY6EOG" target="blank">Keith Devlin - Mathematics - The science of patterns</a></li>
    <li><a href="https://www.amazon.com/Baby-Human-Geniuses-in-Diapers/dp/B000127IG6" target="blank">Baby Human - Genius in Diaper 1</a></li>
    <li><a href="http://amzn.to/2zeH5f0" target="blank">Baby Human - Genius in Diaper 2</a></li>
    <li><a href="https://earlyarts.co.uk/blog/creativity-in-early-brain-development" target="blank">Creativity in Early Brain Development</a></li>
    <li><a href="http://dm16174grt2cj.cloudfront.net/Downloads_Research/young_brains.pdf" target="blank">Young Brains</a></li>
    <li><a href="https://www.kidsclubchildcare.com.au/the-science-of-early-childhood-development/" target="blank">Brains Are Build Not Born</a></li>
    <li><a href="http://www.blue1647.com" target="blank">Blue1647</a></li>
    <li><a href="http://www.imdb.com/title/tt1587359/" target="blank">Top Secret Rosies</a></li>
    <li><a href="https://www.ted.com/playlists/171/the_most_popular_talks_of_all" target="blank">Do Schools Kill Creativity?</a></li>
  </div>
</section>