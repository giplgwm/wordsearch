# Python word search generator

-----
**Lets** start with a simple class with a constructor:
   ```python
   class Wordsearch:

   def __init__(self, size: int):
     self.size = size
     self.grid = [['0' for x in range(size)] for x in range(size)]
     self.hidden_words = {}
   ```
For now, our word search simply takes in a size and creates a grid filled with 0s. We also create an empty hash table so that we can keep track of the words hidden in our grid later. Now what does this class need? We need to be able to add a list of words to the puzzle. They should be randomly added horizontally, vertically, or diagonally. Words should be able to be inserted backwards, and use letters from other words.

-----

Next, it would be nice to be able to print() our wordsearch object, so lets define its __str__() method:
   ```python
   def __str__(self):
     grid_str = ''
     for row in self.grid:
       for letter in row:
         grid_str += f'{letter} '
       grid_str += '\n'
     return grid_str
   ```
This will make it much easier to work with these objects as we move on.

-----

### Adding words to the puzzle

**From** a high level view, we can define adding a wordlist to our puzzle as:
  ```python
  def add_wordlist(self, wordlist):
    successfully_added = []
    for word in wordlist:
      if self._addword(word): #If the function returns true
        self.hidden_words.append(word) #Add the word to our puzzle's word list
        successfully_added.append(word) #Add the word to our return value
    self._fill() #After we add the word list, fill the gaps with random letters
    return successfully_added
  ```
  
I know that eventually we want `self._addword()` to randomly insert each word in a different orientation, but for now lets focus on solving the problem just for inserting a word horiontally into a puzzle. Hopefully our solution to this will give us some insight into how to implement the other orientations:
    
   ```python
   def _add_horizontal(self, word):
     word_len = len(word)
     rows_with_space = self._find_rows_with_gap(word, self.grid)
     row_indexes = list(rows_with_space)
     if len(row_indexes) == 0:
       return False
     elif len(row_indexes) == 1:
       chosen_row = row_indexes[0]
     else:
       chosen_row = random.choice(row_indexes)
     row_index, gap_start, gap_end = rows_with_space[chosen_row]
     #print(f'ADDING {word} TO ROW {row_index} WITH GAP START {gap_start} AND GAP END {gap_end}')
     #Leaving this print statement indefinitely because its proven extremely useful for several different bugs already, and who knows. try this if this area breaks.
     if gap_end - (word_len - 1) == gap_start:
       starting_index = gap_start
     else:
       starting_index = random.randint(gap_start, gap_end - (word_len - 1))
     for letter in word:
       self.grid[row_index][starting_index] = letter
       starting_index += 1
     return True
   ```
This function creates a dictionary (and a list) of rows that can fit our word in it (that is, words who have a "gap" of placeholder values long enough to insert our word), by calling `self._find_rows_with_gap()`. The only problem is, we dont have that function yet. For now, we can pretend we have a function that returns a dictionary of rows that have space for our word, and outline the next steps our code should take, then we can create the function to support it after.
   
Once we have our dictionary, we check the length of it. If its 0, there isn't space to insert the word this way so we return false. If there is 1 space where the word can fit, we choose the only element in our list. If there are more options, we choose one randomly. This check got added due to errors being thrown when random.randint(x, y) only has one value to choose from, so we have to handle that case manually.
   
Our next question seems to be "Where to we place the word?". Consider that the gap will be at *at least* as long as our word, but may very well be longer. So we calculate viable spaces for the first letter (spaces where there is still a gap that fits the remainder of our word), and then make that our starting index in the row.
   
Once we choose a starting index, we iterate through our word and insert the letter at each position, then move over one space in the column to insert the next word. We return True to signify the word was indeed added and should be appended to the list of words hidden in the puzzle!

-----

**Now... how do we find the rows with a suitable gap to fit our word?** Lets look at the function I came up with:
   ````python
   def _find_rows_with_gap(self, word: str, grid):
     rows = {}
     length = len(word)
     for index, row in enumerate(grid):
       max_gap = 0
       max_start = -9
       current_gap = 0
       current_start = -9 
       for char_index, char in enumerate(row):
         if char == '0':  # or word[current_gap+1] == char: IMPLEMENT FOR GOODNESS SAKE
           current_gap += 1
           if current_start == -9:  #If we haven't yet started a 'gap'
             current_start = char_index  #This is our start
         else:
           current_gap = 0
           if current_start != -9:  #If this is the end of our gap
             current_start = -9
         if current_gap > max_gap:
           max_gap = current_gap
           max_start = current_start

       if max_gap >= length:
         rows[index] = (index, max_start, max_start + max_gap - 1)
     return rows
   ````
   Lets break this down.
   
* For each row in the grid:
  * At the beginning of the row, we initialize our max_gap and current_gap to 0, and our max_start and current_start (we will use these to mark the starting position of our "gap" so we can find the gap again later to insert the word) are set to negative values for now to distinguish them from genuine index values.
     
  * For each character in the row, if it is a '0', our current_gap is 1 space larger. If we do not have a start_index for our current gap yet, this characters position is that start index.
     
  * If the character is a letter, our gap has ended. If we have a value for our current_start value, reset it to -9 until we find another placeholder character.
   
  * If our current_gap is larger than the max_gap we had found before, this gap is our max_gap and the starting point of this gap is the starting point of our max gap.
   
  * Finally, once we know how large of a gap is in the row we check if it is long enough for our word to fit and if so we add it to our list.
   
* At the end of our function we return a list of rows with suitable gaps in them, as well as where that gap begins in the row! Just what our `_add_horizontal()` function needs!

Notice this line where we check if our char is a placeholder:

  > if char == '0':

and the comment immediately after:
  
  > or word[current_gap+1] == char: IMPLEMENT FOR GOODNESS SAKE

At that point I had begun to see how my gap-finding solution wasn't the best one to the problem - since it didnt account for letters that weren't placeholders, but would still allow for the word to be inserterted. For example, consider trying to place the word "Hats" in a gap that looks like _ _ _ S. This should clearly be possible, but our algorithm above will not see it. Unfortunately I had already spent enough time on this function and decided to stick with it and maybe refactor later. 

For now my solution to adding words in every orientation, as well as connecting to letters from other words is going to have 4 functions:
  1. `_add_horizontal()` which we implemented above. This will serve as the model for how we solve this problem, with 3 slightly different implementations.
  2. `_add_vertical()` which will insert words... vertically.
  3. `_add_diagonal()`
  4. `add_to_existing()` whcih will be the 'bandage' solution to not implementing letter-sharing in the original solution. If we just add this as an option along with the other orientations, we can achieve the same end-result.
  5. 
  Having 4 similar functions seems like a design issue to me, but again, I'm not trying to write the best word search library in python, I just need it to work consistently for now. *To be fair, if my original solution was better we may only need 3 functions here, one for each orientation.* Having this `_add_to_existing()` function does give us some interesting possibilities later on in 'how' exactly we generate the puzzle. for example, a puzzle where every word tries to start from an existing one first will look very different from one where each word first tries to insert vertically/horizontally first. We could potentially even extend this into a pattern generator by mixing our functions differently.

-----

### Vertical words

Moving on, our `_add_vertical()` function has an interesting solution. Instead of rewriting the previous solution completely, lets try to reuse what we can. Take a look at the first few lines of `_add_vertical()` to see what I mean:
  ```python
  def _add_vertical(self, word):
    word_len = len(word)
    grid = self._transposed_grid()
    cols_with_spaces = self._find_rows_with_gap(word, grid)
  ```

Instead of making a new function to find columns with gaps, we can actually transpose our grid so that our columns **are** rows, and then just use the function we *already* made to find the gaps in those rows! This was a satisfying revelation to come to, and im glad I kept looking for a better answer than rewriting both functions. The rest of this function looks identical to `_add_horizontal()` aside from the actual insertion logic, which seems to be a sign that this could be arranged in a more DRY way, but it'll do for now. 

Here is the complete function:

  ```python
  def _add_vertical(self, word):
    word_len = len(word)
    grid = self._transposed_grid()
    cols_with_spaces = self._find_rows_with_gap(word, grid)
    col_indexes = list(cols_with_spaces)
    if len(col_indexes) == 0:
      return False
    elif len(col_indexes) == 1:
      chosen_col = col_indexes[0]
    else:
      chosen_col = random.choice(col_indexes)
    col_index, gap_start, gap_end = cols_with_spaces[chosen_col]

    if gap_end - (word_len - 1) == gap_start:
      starting_index = gap_start
    else:
      starting_index = random.randint(gap_start, gap_end - (word_len - 1))

    for letter in word:
      self.grid[starting_index][col_index] = letter
      starting_index += 1
    return True
  ```

  -----

### Diagonal Words

Our next problem is adding words diagonally. Fundamentally, this is more complex than simply adding to grids or rows directly. In the previous 2 problems, we had to analyze 1 row or column at a time, but to do diagonal insertions we will need to inspect spaces from multiple columns/rows at once. 

Tabbing over to a virtual whiteboard, here was my thought process to solve this:

Take a grid with size 4:

<img src='images/empty_grid.png' width='40%'>

After staring into these empty circles for a while, I started to see the simple pattern that would lead to an answer here.

1. For a given start node, we can make a right diagonal with the starting node + every right child after it

2. For a given start node, we can make a left diagonal with the starting node + every left child after it

<img src='images/columns.png' width='40%'>

So for solving our problem of finding diagonals that can fit our word, here are the questions we need to answer:
  * ~How do we find diagonals?~ - We defined a diagonal above as our starting node + all children diagonally to one side.
  * Which nodes do we start a diagonal from?
  * How do we get the next node in a diagonal? Aka our right or left child as called above.

Lets look at that last question. If we take an arbitrary node from our grid, how do we get the node down and to the right of it?

<img src='images/arbitrary_selection.png' width='40%'>

Lets highlight the "right child" and label the rows and columns so we can look for a pattern

<img src='images/arbitrary_selection_right_child.png' width='40%'>

We can see that our Node is in row 1, column 2. Looking at its right child, that sits in row 2, column 3. If that Node had a right child, it would be in row 3, column 4.

The pattern here is simple. **For a node in Row x and column y, its right child will be in row x+1 and column y+1.**

<img src='images/right_child_general.png' width='40%'>

Now lets look at the left child of our node and see if we can figure out that pattern:

<img src='images/arbitrary_selection_left_child.png' width='40%'>

We see our Node is still in row 1, column 2. Now our child is in row 2, column 1 though!
If we look at that node's left child, we can see it sits in row 3, column 0.

We can extract the general rule: **For a node in Row x and column y, its right child will be in row x+1 and column y-1.**

<img src='images/children_general.png' width='40%'>

Back to our list of questions:

* ~How do we find diagonals?~ - We defined a diagonal above as our starting node + all children diagonally to one side.
* Which nodes do we start a diagonal from?
* ~How do we get the next node in a diagonal?~ We can use our formula above to get the diagonal left or right child of any node

So now the only piece we're missing to be able to find every diagonal in the grid is **Which nodes do we start a diagonal from**? Intuitively, we can say that we should only start a diagonal from a node that sits on the edge of the grid.

<img src='images/edge_highlighted.png' width='40%'>

With a bit more thinking, we see that the bottom row actually can be omitted because there will never be a child in the row beneath, since there will *never* be a row underneath our final one. So the only valid starting positions for a diagonal in our grid are:

<img src='images/diagonal_starts.png' width='40%'>

-----

Now that our 3 questions are answered, we can begin implementing our solution for diagonals! Lets start with a function that returns a list of all the diagonals in our grid. My initial plan was to be able to send this list into our original `_find_rows_with_gap()` function but we'll see why that won't work later.

  ```python
  def _transpose_diagonal(self):
    grid_transposed = []
    visited = {}
    directions = {}
    current_node = (self.size - 2, 0)
    while current_node:
      visited[current_node] = True
      diagonal_right = self._diag_right(current_node)
      diagonal_left = self._diag_left(current_node)
      if diagonal_right:
        directions[len(grid_transposed)] = 'R'
        grid_transposed.append(diagonal_right)
      if diagonal_left:
        directions[len(grid_transposed)] = 'L'
        grid_transposed.append(diagonal_left)
      current_node = self._next_diag_start(current_node)
    return grid_transposed, directions
  ```

Lets walk through this:

1. We initialize a new list for our diagonals, A dictionary for the nodes we've already visited and one for the directions of each diagonal in our main list (this will be important later!)
2. We set our current_node to row: size - 2, column: 0
   This is the bottom left highlighted node in our image. We will start here and traverse the outer edge of our grid until we have analyzed each diagonal.
3. While we have a current_node:
    * We set the value of our node in the visited dictionary to True, so we can reference that later when deciding which node to go to next and ensure we never get caught in a traversal loop.
    * We set a variable named diagonal_right to the return value of a function that generates a right diagonal with our strategy above!
    * We do the same with a variable named diagonal_left and a slightly different function to get the diagonal to the left.
    * If either of these columns does not exist (for example, nodes along the left edge will not have a left diagonal), our function returns false. So we check if either of these values are 'truthy' (aka not None or False or an equivalent.) and if so, append them to our list of diagonals. We also add an entry to our directions dict so that we know later this is a right diagonal.
    * Our next current_node is determined by passing in the current_node to a function that uses the pattern we analyzed earlier to pick the next node we should use as a start.
4. We return our diagonals, and the matching dictionary of their directions.

Of course, we're relying on some functions that we have not created yet, but thats ok. lets make them now:

  ```python
  def _diag_right(self, node, letters=None):
    if node[1] == self.size - 1:
      return None  # We will never have a right diag child starting from the right edge
    if letters is None:
      letters = []
      letter = self.grid[node[0]][node[1]]
      letters.append((letter, node[0], node[1]))
    try:
      child = self.grid[node[0] + 1][node[1] + 1]
    except IndexError:
      return
    else:
      letters.append((child, node[0] + 1, node[1] + 1))
      self._diag_right((node[0] + 1, node[1] + 1), letters)
    if len(letters) == 1:
      letters = None
    return letters
  ```
In this function we:
* Check if the node is along the right edge, if so return None. A node on the right edge can not have a right diagonal child. This is a base case.
* If this is our first recursive call to the function, set letters to an empty list. This will keep track of the letters in our diagonal, being added to by each recursive call and finally returned. We also add our initial node's letter to the list here.
* We try to set `child` to the right child of our node using the formula from earlier. If this throws an IndexError, the child does not exist and we can just return for now. This is one base case of our recursive function, when we are not on the right edge but there is still no right child (at the bottom of the grid perhaps)
* If no exception is thrown when we access the right child, add its letter to our list! Now recursively call this function on the right child to get it's right child, until we hit the right edge and return.
* If the length of our diagonal is 1, we do not have a diagonal here. return letters, which will either have our diagonal or None - as our previous function expected.

The `_diag_left()` function is the same, just with the cordinate of the child calculated differently, using the patterns observed previously.

We also used `_next_diag_start()` in our function earlier, so we need to define that as well:

  ```python
  def _next_diag_start(self, node):
    if node[1] == 0:  #If we're along the left edge of the grid, we either need a neighbor to the top or right
      if node[0] == 0:  #If we're in the top left corner, go right!
        return (node[0], node[1] + 1)
      else:  #Otherwise, go up!
        return (node[0] - 1, node[1])
    if node[0] == 0:  #If we're along the top edge of the grid, we either need a neighbor to the right or below
      if node[1] == self.size - 1:  #If we're in the top right corner, go down!
        return (node[0] + 1, node[1])
      else:  #Otherwise, go to the right!
        return (node[0], node[1] + 1)
    if node[1] == self.size - 1:  # If we're along the bottom edge of the grid we need a neighbor below
      if node[0] + 1 > self.size - 2:  #If we're at the last node in our U shape
        return None
      else:
        return (node[0] + 1, node[1])
  ```
This series of if-else statements just determines which edge we're currently on, and selects the next node accordingly. When we hit a corner, we switch directions. We return the coordinates for the next node on the edge. If we are at the last node in our pattern, return None to let the calling function know to end.

Now we can work on our main `_add_diagonal()` function. Unfortunately though, there is an issue with find_rows_with_gap that will stop us from reusing it this time. We took for granted that we were already iterating through a given column or row before, so the function only needed to return the start location's index *within that row/column* now though, we need our function to return a row **and** column for us to be able to find the starting position. For this, we'll define a new function called `_find_diags_with_gap()` to help.

  ```python
  def _add_diagonal(self, word):
    word_len = len(word)
    grid, directions = self._transpose_diagonal()
    diags_with_space = self._find_diags_with_gap(word, grid)
  ```
Here is that function:

  ```python
  def _find_diags_with_gap(self, word: str, grid):
    diags = {}
    length = len(word)
    for index, row in enumerate(grid):
      max_gap = 0
      max_start = -9
      current_gap = 0
      current_start = -9  
      for (char, row, col) in row:
        if char == '0': 
          current_gap += 1
          if current_start == -9:  #If we haven't yet started a 'gap'
            current_start = (row, col)  #This is our start
        else:
          current_gap = 0
          if current_start != -9:  #If this is the end of our gap
            current_start = -9
        if current_gap > max_gap:
          max_gap = current_gap
          max_start = current_start
      if max_gap >= length:
        diags[index] = (index, max_start, max_gap)
    return diags
  ```
Notice it's just a modified version of our earlier function, but with specificly returned results to help our new goal. I even left the comments when I copy/pasted it from above. This is a common pattern for me when writing larger programs that may need to solve several similar problems. To work on not replicating my code so much, the next book i'll be studying is "Clean Code" by Robert C Martin. Hopefully it helps. 

Now that we have that function in place, we can actually insert our word! Lets look at the complete function for diagonal insertions:

  ```python
  def _add_diagonal(self, word):
    word_len = len(word)
    grid, directions = self._transpose_diagonal()
    diags_with_space = self._find_diags_with_gap(word, grid)
    options = list(diags_with_space)
    if len(options) == 0:
      return False
    elif len(options) == 1:
      chosen_diag = options[0]
    else:
      chosen_diag = random.choice(options)
    direction = directions[chosen_diag]
    index, start, gap_size = diags_with_space[chosen_diag]
    if direction == 'L':
      nodes = [(start[0] + x, start[1] - x) for x in range(gap_size)]
    else:
      nodes = [(start[0] + x, start[1] + x) for x in range(gap_size)]
    if len(nodes) - (word_len - 1) == 1:
      starting_node = 0
    else:
      starting_node = random.randint(0, len(nodes) - (word_len - 1) - 1)
    for letter in word:
      row = nodes[starting_node][0]
      col = nodes[starting_node][1]
      self.grid[row][col] = letter
      starting_node += 1
    return True
  ```
This insertion is a bit more complicated, since we have to account for the direction of the diagonal and find the nodes involved in each diagonal. But it follows most of the same logic as the previous functions and should be easy enough to follow.

At this point we have a word search class that can insert words in 3 orientations! Now lets make our main functions for inserting a word and a word list. We can group them together here since the logic is relatively simple:

  ```python
  def add_wordlist(self, wordlist):
    successfully_added = []
    for word in wordlist:
      if self._addword(word):
        self.hidden_words.append(word)
        successfully_added.append(word)
    self._fill() #Not implemented yet! see below
    return successfully_added

  def _addword(self, word):
    if len(word) > self.size:
      return False
      
    placements = "hvd"
    placement = random.choice(placements)
    direction = random.choice('fffbf')
    
    if direction == 'b':
      word = word[::-1]
      
    successful = False
    
    while not successful:
      if placement == 'h':
        successful = self._add_horizontal(word)
      elif placement == 'v':
        successful = self._add_vertical(word)
      elif placement == 'd':
        successful = self._add_diagonal(word)

      tries += 1
      placements.replace(placement, '')
      if len(placments) > 1:
        placement = random.choice(placements)
      else:
        placement = placements
    return successful
  ```
Our function simply takes every word in the list, passes it to `_addword()` and if that function returns a truthy value, adds the word to our lists to keep track that it was actually added to the puzzle. In a case where a word fails to be added False will be returned and we will simply continue down the list.

Behind the scenes, `_addword()` simply uses random.choice() to decide which placement option to try, and if it fails will try the next ones continuously before giving up. it also uses random.choice to determine if a word should be added forwards or backwards and uses string slicing to pass in the backwards word if chosen, giving us a simple solution to one of our goals stated at the beginning.

Once we add our word, we will want the puzzle to be filled in with random letters. Thats simple to implement:

  ```python
  def _fill(self):
    for index, row in enumerate(self.grid):
      for char_index, char in enumerate(row):
        if char == '0':
          self.grid[index][char_index] = random.choice('abcdefghijklmnopqrstuvwxyz')
  ```

Now the only thing missing is to use existing letters to add a word. Currently we can add words in multiple orientations but only in places where an empty gap exists. We'd like to be able to build off of existing words to add more.

-----

I gave this problem a solid hour or 2 of pondering before reaching for ChapGPT to give some guidance. After playing with the problem for a bit this is what I had:

  ```python
  def _add_word_from_existing(self, word):
    word_len = len(word)
    rows_with_letters = {}
    for row_index, row in enumerate(self.grid):
      for col_index, char in enumerate(row):
        if char in word:
          rows_with_letters[(row_index, col_index)] = char

    for (row, col), letter in rows_with_letters.items():
      neighbors = self._find_suitable_neighbors(row, col, word)
    ...


  def _find_suitable_neighbors(self, row, col, word):
    directions = [(0, 1), (0, -1), (1, 0), (-1, 0), (1, 1), (1, -1), (-1, 1),
                  (-1, -1)]
    ...
  ```
I had a basic idea of how I might be able to solve the problem, but i was still pretty far from an entire solution. Explaining what i wanted to do to chatGPT and providing the code I had so far it gave me this working solution:

  ```python
  def _add_word_from_existing(self, word):
    word_len = len(word)
    rows_with_letters = {}
    for row_index, row in enumerate(self.grid):
      for col_index, char in enumerate(row):
        if char in word:
          rows_with_letters[(row_index, col_index)] = char

    for (row, col), letter in rows_with_letters.items():
      neighbors = self._find_suitable_neighbors(row, col, word)
      if neighbors:
        self._place_word(neighbors, word)
        return True

    return False

  def _find_suitable_neighbors(self, row, col, word):
    directions = [(0, 1), (0, -1), (1, 0), (-1, 0), (1, 1), (1, -1), (-1, 1),
                  (-1, -1)]
    for dx, dy in directions:
      if self._can_place_word(row, col, dx, dy, word):
        return (row, col, dx, dy)
    return None

  def _can_place_word(self, row, col, dx, dy, word):
    word_len = len(word)
    for i in range(word_len):
      new_row = row + i * dx
      new_col = col + i * dy
      if not (0 <= new_row < self.size and 0 <= new_col < self.size):
        return False
      if self.grid[new_row][new_col] not in ['0', word[i]]:
        return False
    return True

  def _place_word(self, start_pos, word):
    row, col, dx, dy = start_pos
    for letter in word:
      self.grid[row][col] = letter
      row += dx
      col += dy
  ```

Looking through this, chatgpt used the basic structure I had for my solution and added some logic and helper functions to make it work. build a dictionary of nodes in the grid that have suitable letters for us to try branching from, then we check if one of those has enough space to fit the word. If so, we place the word there and return. If for some reason we can not fit the word, we eventually return False.

Surprisingly enough, chatgpt spit out a working solution on the first prompt - and i decided to keep it. The main goal of this project is not the Wordsearch class, and I can personally accept if i couldnt solve every challenge about this project on my own this time around. I can study more on relevant algorithms and work on solving problems on that level at my leisure.

Now that we have a new function for adding words, we can make a minor change to our `_add_word()` function, adding an option to 'placements' for e and an else statement for that case:
  ```python
  def _addword(self, word):
    if len(word) > self.size:
      return False
    placements = "hvde"
    placement = random.choice(placements)
    direction = random.choice('fffbf')
    if direction == 'b':
      word = word[::-1]
    successful = False
    while not successful:
      if placement == 'h':
        successful = self._add_horizontal(word)
      elif placement == 'v':
        successful = self._add_vertical(word)
      elif placement == 'd':
        successful = self._add_diagonal(word)
      elif placement == 'e':
        successful = self._add_word_from_existing(word)
      placements.replace(placement, '')
      if len(placments) > 1:
        placement = random.choice(placements)
      else:
        placement = placements
    return successful
  ```
For now we've accomplished everything we set out to do, so that is where we'll leave our Wordsearch class. If we need more functionality from it later we can come back.

----