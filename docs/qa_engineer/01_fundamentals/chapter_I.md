# Chapter I - Programming Exercises
`(avr. time for this chapter: 2 days)`

Before diving into testing frameworks, it's essential to have strong programming fundamentals. These exercises will help you develop problem-solving skills that are crucial for writing effective tests and debugging issues.

This chapter contains a battery of exercises covering string manipulation, array operations, and algorithmic thinking.

## String Manipulation Exercises

### Exercise 1: Count Word Occurrences

Write a function that counts how many times each word appears in a sentence.

**Requirements:**
- Input: A string sentence
- Output: An object/dictionary with words as keys and counts as values
- Words should be case-insensitive
- Ignore punctuation

**Example:**
```javascript
// Input
"The quick brown fox jumps over the lazy dog. The dog was not amused."

// Output
{
  "the": 3,
  "quick": 1,
  "brown": 1,
  "fox": 1,
  "jumps": 1,
  "over": 1,
  "lazy": 1,
  "dog": 2,
  "was": 1,
  "not": 1,
  "amused": 1
}
```

**Steps to implement:**

1. Create a function `countWords(sentence)`
2. Convert the sentence to lowercase
3. Remove punctuation using regex
4. Split the sentence into words
5. Iterate through words and count occurrences
6. Return the result object

**JavaScript Solution Template:**
```javascript
function countWords(sentence) {
  // Your implementation here
}

// Test cases
console.log(countWords("Hello hello HELLO"));
// Expected: { "hello": 3 }

console.log(countWords("The cat sat on the mat. The cat was happy."));
// Expected: { "the": 3, "cat": 2, "sat": 1, "on": 1, "mat": 1, "was": 1, "happy": 1 }
```

**Python Solution Template:**
```python
def count_words(sentence):
    # Your implementation here
    pass

# Test cases
print(count_words("Hello hello HELLO"))
# Expected: {"hello": 3}

print(count_words("The cat sat on the mat. The cat was happy."))
# Expected: {"the": 3, "cat": 2, "sat": 1, "on": 1, "mat": 1, "was": 1, "happy": 1}
```

---

### Exercise 2: Find Most Repeated Word

Write a function that finds the most frequently occurring word in a sentence.

**Requirements:**
- Input: A string sentence
- Output: The most repeated word (if tie, return any one)
- Case-insensitive comparison

**Example:**
```javascript
// Input
"To be or not to be, that is the question. To be is everything."

// Output
"to" // appears 3 times
```

**Steps to implement:**

1. Use your `countWords` function from Exercise 1
2. Find the word with the maximum count
3. Return that word

---

### Exercise 3: Find Duplicate Words

Write a function that returns all words that appear more than once.

**Requirements:**
- Input: A string sentence
- Output: An array of words that appear more than once
- Sorted alphabetically

**Example:**
```javascript
// Input
"The quick brown fox jumps over the lazy brown dog"

// Output
["brown", "the"]
```

---

### Exercise 4: Reverse Words in Sentence

Write a function that reverses the order of words in a sentence while keeping each word intact.

**Example:**
```javascript
// Input
"Hello World"

// Output
"World Hello"
```

---

## Array Manipulation Exercises

### Exercise 5: Remove Words from Array

Write a function that removes specific words from an array.

**Requirements:**
- Input: An array of words and an array of words to remove
- Output: A new array without the removed words
- Original array should not be modified

**Example:**
```javascript
// Input
words = ["apple", "banana", "cherry", "date", "elderberry"]
toRemove = ["banana", "date"]

// Output
["apple", "cherry", "elderberry"]
```

**JavaScript Solution Template:**
```javascript
function removeWords(words, toRemove) {
  // Your implementation here
}

// Test cases
console.log(removeWords(["a", "b", "c", "d"], ["b", "d"]));
// Expected: ["a", "c"]

console.log(removeWords(["hello", "world", "foo", "bar"], ["foo"]));
// Expected: ["hello", "world", "bar"]
```

**Python Solution Template:**
```python
def remove_words(words, to_remove):
    # Your implementation here
    pass

# Test cases
print(remove_words(["a", "b", "c", "d"], ["b", "d"]))
# Expected: ["a", "c"]

print(remove_words(["hello", "world", "foo", "bar"], ["foo"]))
# Expected: ["hello", "world", "bar"]
```

---

### Exercise 6: Remove Duplicates from Array

Write a function that removes duplicate values from an array while preserving order.

**Example:**
```javascript
// Input
[1, 2, 2, 3, 4, 4, 5, 1]

// Output
[1, 2, 3, 4, 5]
```

---

### Exercise 7: Find Common Elements

Write a function that finds common elements between two arrays.

**Example:**
```javascript
// Input
arr1 = [1, 2, 3, 4, 5]
arr2 = [4, 5, 6, 7, 8]

// Output
[4, 5]
```

---

### Exercise 8: Flatten Nested Array

Write a function that flattens a nested array to a single level.

**Example:**
```javascript
// Input
[1, [2, 3], [4, [5, 6]], 7]

// Output
[1, 2, 3, 4, 5, 6, 7]
```

---

### Exercise 9: Group Array Elements

Write a function that groups array elements by a given property.

**Example:**
```javascript
// Input
const users = [
  { name: "Alice", role: "admin" },
  { name: "Bob", role: "user" },
  { name: "Charlie", role: "admin" },
  { name: "David", role: "user" }
];

groupBy(users, "role");

// Output
{
  "admin": [
    { name: "Alice", role: "admin" },
    { name: "Charlie", role: "admin" }
  ],
  "user": [
    { name: "Bob", role: "user" },
    { name: "David", role: "user" }
  ]
}
```

---

### Exercise 10: Chunk Array

Write a function that splits an array into chunks of a specified size.

**Example:**
```javascript
// Input
arr = [1, 2, 3, 4, 5, 6, 7, 8]
size = 3

// Output
[[1, 2, 3], [4, 5, 6], [7, 8]]
```

---

## Algorithm Exercises

### Exercise 11: Two Sum

Write a function that finds two numbers in an array that add up to a target sum.

**Example:**
```javascript
// Input
nums = [2, 7, 11, 15]
target = 9

// Output
[0, 1] // because nums[0] + nums[1] = 2 + 7 = 9
```

---

### Exercise 12: Valid Palindrome

Write a function that checks if a string is a palindrome (ignoring spaces and punctuation).

**Example:**
```javascript
// Input
"A man, a plan, a canal: Panama"

// Output
true
```

---

### Exercise 13: FizzBuzz

Write a function that returns an array of numbers from 1 to n, but:
- For multiples of 3, use "Fizz"
- For multiples of 5, use "Buzz"
- For multiples of both, use "FizzBuzz"

**Example:**
```javascript
// Input
n = 15

// Output
[1, 2, "Fizz", 4, "Buzz", "Fizz", 7, 8, "Fizz", "Buzz", 11, "Fizz", 13, 14, "FizzBuzz"]
```

---

### Exercise 14: Anagram Check

Write a function that checks if two strings are anagrams of each other.

**Example:**
```javascript
// Input
str1 = "listen"
str2 = "silent"

// Output
true
```

---

### Exercise 15: Find Missing Number

Write a function that finds the missing number in an array containing n distinct numbers from 0 to n.

**Example:**
```javascript
// Input
[3, 0, 1]

// Output
2
```

---

## Submission Guidelines

For each exercise:

1. Write your solution in both JavaScript and Python (if comfortable with both)
2. Include at least 3 test cases per function
3. Consider edge cases:
   - Empty inputs
   - Single element inputs
   - Large inputs
4. Add comments explaining your approach
5. Analyze the time and space complexity of your solution

## Self-Assessment

After completing these exercises, you should be able to:

- [ ] Manipulate strings efficiently
- [ ] Work with arrays and common operations
- [ ] Understand basic algorithmic patterns
- [ ] Write clean, readable code
- [ ] Think about edge cases
- [ ] Analyze time and space complexity

## Next Steps

Once you've completed these exercises, move on to [Chapter II - Testing Fundamentals](chapter_II.md) to learn the core concepts of software testing.
