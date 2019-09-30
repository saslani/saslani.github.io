---
layout: post
title:  "Getting Started with Ruby: Tic Tac Toe"
date:   2016-10-23 22:53:39
categories: Ruby
---

This program is designed for those who are want to learn Ruby by doing small project that helps them build their skills.
Below is the link to my workshop, I encourage you to use [repl.it](repl.it) since it is the easiest, not necessarily the best, way to get started with ruby. You need to sign up, create a session, choose ruby from the list, and start your work.

Note: I found out that sometimes after creating a new session I have to refresh my browser in order to see my new session.

![](/img/ruby_beginners/repl.gif)

Another way to do this exercise is to use a simple text file and save it in your development directory, or Desktop or where ever you prefer to work. You can name it 'tic_tac_toe.rb', the name dosn't matter however, we prefer underscored lowercase names but the extension has to be '.rb'. Then from your terminal navigate to the directory that contains your file. Use your IDE of choice to go through the steps. In order to run your program use 'ruby tic_tac_toe.rb' from your terminal.

![](/img/ruby_beginners/tic_tac_toe.gif)

Have fun and feel free to shoot me an email if you have any questions!

<iframe src="//slides.com/sarahaslanifar/tic-tac-toe/embed" width="700" height="500" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

Following is the step-by-step solution to the tic-tac-toe beginner; try not to use this unless you are really stuck!


Step 1:

{% highlight ruby %}
################
#	LET'S PLAY
################
# Here is the order in which we want to play the game. Run the code section below and see what happens
# Note:
# You will see the following error:
# -> undefined local variable or method `initialize_board` for main:Object
# This means the repl doesn't understand 'initialize_board', let's go ahead and define this method. Then go back up to Step 2 and build your tic-tac-toe

initialize_board
begin
  break if !play
  switch_player
end while !game_over?
puts "GAME OVER" if(game_over?)
{% endhighlight %}

----------

Step 2:

{% highlight ruby %}
def initialize_board
end
{% endhighlight %}
----------

Step 3:

{% highlight ruby %}
def play
end
{% endhighlight %}

----------

Step 4:

{% highlight ruby %}
def switch_player
end
{% endhighlight %}

----------

Step 5:

{% highlight ruby %}
def game_over?
end
{% endhighlight %}

By the end of Step 5, your program looks like this:

{% highlight ruby %}
def initialize_board
end

def play
end

def switch_player
end

def game_over?
end

initialize_board
begin
break if !play
switch_player
end while !game_over?
puts "GAME OVER" if(game_over?)
{% endhighlight %}

----------

Step 6:

{% highlight ruby %}
def initialize_board
  @board = Array.new(3){ Array.new(3, " ") }
end

def print_board
  puts "---------"
  @board.each do |row|
    puts row.join(" | ")
    puts "---------"
  end
end

# .
# .

initialize_board
print_board
begin
  break if !play
  switch_player
end while !game_over?
puts "GAME OVER" if(game_over?)
{% endhighlight %}

----------

Step 7:

{% highlight ruby %}
PLAYER_ONE = 'X'
PLAYER_TWO = 'O'

def initialize_board
  @player = PLAYER_ONE
  @board = Array.new(3){ Array.new(3, " ") }
end

def get_inputs
  print "#{@player}, please enter the row: "
  input_row = gets

  print "#{@player}, please enter the column: "
  input_column = gets

  [input_row.to_i, input_column.to_i]
end

def play
  inputs = get_inputs
end

def game_over?
  true
end
{% endhighlight %}

----------

Step 8:

{% highlight ruby %}

def update_board(inputs)
  @board[inputs[0]-1][inputs[1].-1] = @player
end

def play
  inputs = get_inputs
  update_board(inputs)
  print_board
end
{% endhighlight %}

----------

Step 9:

{% highlight ruby %}
def initialize_board
  @count = 9
  @player = PLAYER_ONE
  @board = Array.new(3){ Array.new(3, " ") }
end

def switch_player
  if(@player == PLAYER_ONE)
    @player = PLAYER_TWO
  else
    @player = PLAYER_ONE
  end
end

def game_over?
  @count = @count - 1
  @count <= 0
end
{% endhighlight %}

----------

Step 10:

{% highlight ruby %}
def get_inputs
  print "#{@player}, please enter the row: "
  input_row = gets
  return false if(input_row.to_i == 999)

  print "#{@player}, please enter the column: "
  input_column = gets
  return false if(input_column.to_i == 999)

  [input_row.to_i, input_column.to_i]
end

def play
  inputs = get_inputs
  return false if !inputs
  update_board(inputs)
  print_board
end
{% endhighlight %}

----------

Step 11: 

{% highlight ruby %}
def get_inputs
  begin
    print "#{@player}, please enter the row: "
    input_row = gets
    return false if(input_row.to_i == 999)

    print "#{@player}, please enter the column: "
    input_column = gets
    return false if(input_column.to_i == 999)

    if(!valid = valid_move?(input_row.to_i, input_column.to_i))
      puts "That one is taken \"#{@player}\", try again: "
    end
  end while(!valid)

  [input_row.to_i, input_column.to_i]
end

def valid_move?(input_row, input_column)
  @board[input_row.-1][input_column.-1].strip.empty?
end
{% endhighlight %}

Finished program:

{% highlight ruby %}
PLAYER_ONE = 'X'
PLAYER_TWO = 'O'

def initialize_board
  @count = 9
  @player = PLAYER_ONE
  @board = Array.new(3){ Array.new(3, " ") }
end

def print_board
  puts "---------"
  @board.each do |row|
    puts row.join(" | ")
    puts "---------"
  end
end

def get_inputs
  begin
    print "#{@player}, please enter the row: "
    input_row = gets
    return false if(input_row.to_i == 999)

    print "#{@player}, please enter the column: "
    input_column = gets
    return false if(input_column.to_i == 999)

    if(!valid = valid_move?(input_row.to_i, input_column.to_i))
      puts "That one is taken \"#{@player}\", try again: "
    end
  end while(!valid)

  [input_row.to_i, input_column.to_i]
end

def valid_move?(input_row, input_column)
  @board[input_row.-1][input_column.-1].strip.empty?
end

def update_board(inputs)
  @board[inputs[0]-1][inputs[1].-1] = @player
end

def play
  inputs = get_inputs
  return false if !inputs
  update_board(inputs)
  print_board
end

def switch_player
  if(@player == PLAYER_ONE)
    @player = PLAYER_TWO
  else
    @player = PLAYER_ONE
  end
end

def game_over?
  @count = @count - 1
  @count <= 0
end

initialize_board
print_board
begin
  break if !play
  switch_player
end while !game_over?
puts "GAME OVER"
{% endhighlight %}