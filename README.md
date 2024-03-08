# Word search
A python class for generating and representing word searches

This was the first project I came up with after finishing [A Common Sense Guide to Data Structures and Algorithms](https://www.oreilly.com/library/view/a-common-sense-guide/9781680508048/). After finishing this class I used it to make [this webapp](http://wordsearch-racer.replit.app).

Read the full writeup of my process solving this problem [here](writeup.md).

## How to use:
```python
from wordsearch import Wordsearch

a = Wordsearch(14) # Size needs to be an integer
a.add_wordlist(your_wordlist) # The word list should be a comma separated string. ex "apple, pineapple, peach, pear, strawberry, watermelon"
a.title = "Fruits"
print(a)
print(a.toDict())
```

## Methods:
- add_wordlist(string)
- toDict(): returns a dict of all below object variables.
  
## Variables:
- size: the grid size. Our word search will be a 'size' x 'size' grid of letters.
- grid: this stores our actual puzzle as a 2d array.
- hidden_words: words successfully added to the puzzle are appended to this list. 
- found_words: this was used for the above-mentioned webapp, but it makes sense to have in the class.
- title: the title of the word search.
