# Chapter III
`(avr. time for this chapter: TBD)`

Certain applications are expected to handle large loads of network traffic and therefore comes the need to test how the system behaves under such conditions.

There are several possible solutions to this problem such as locking the database when a record is being updated or establishing a queue to process requests avoiding concurrency altogether.

In this chapter, we'll focus on the most common solution for concurrency tests, locking database operations when a transaction is uncommitted.

## Preparation

Let's imagine our e-book shop decides to cater to people who appreciate a physical copy of a book.

Physical copies require a stock value which can never go below 0.

A user may also want to purchase more than 1 physical copy at a time, which is impossible for e-books.

Make the necessary changes to the project you see fit to implement this feature.

## Testing

[Threading](https://guides.rubyonrails.org/threading_and_code_execution.html) is a great way to implement concurrency in specs.

Create any tests you see fit to test physical book purchases with the system under the stress of multiple requests.