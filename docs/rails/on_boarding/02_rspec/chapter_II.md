# Chapter II
`(avr. time for this chapter: TBD)`

While creating specs, you may encounter certain constraints that limit the capabilities of RSpec. For example, these constraints could include factors like testing time limitations or the need to handle external API requests.

In this chapter, you'll get to know some gems that can be integrated with RSpec to overcome such limitations.

## Test Doubles

[Test Doubles](https://rspec.info/features/3-13/rspec-mocks/basics/test-doubles/) allow you create mockups of objects to use in your tests.

They are useful as you can define the return value of each method you define, allowing you to get the outcome you desire without calling the original Object, which might have additional logic you do not wish to take into account.

This mockups will raise an error when any method that you did not define is called.

If you have a service which requires multiple parameters or additional context to work properly, you can create a double of it and define exactly what it will return. This is assuming you already have specs for the service in question.

## Partial Doubles

[Partial Doubles](https://rspec.info/features/3-13/rspec-mocks/basics/partial-test-doubles/) allow you to control the behavior of a single method as opposed to making an entire mockup object.

This can be useful when you just need to change the output of a single method.

## Spies

[Spies](https://rspec.info/features/3-13/rspec-mocks/basics/spies/) Are a type of double that can validate if the object received a call to a specific method.

Both Test and Partial doubles can be used as a spy. This allows you to create specs around if certain methods were called.

## VCR

External API requests can often be a liability for your specs. You can't control the data they provide and any changes made to its response contents may break your specs.

[VCR](https://github.com/vcr/vcr) is a gem which allows you to record HTTP interactions and reply them during future test runs for fast, deterministic, accurate tests.

The code snippet below replicates an external API call designed to retrieve book reviews.

Add it to your Books Controller and then create the necessary specs you seem fit to test this method.

> After the inclusion of the provided code snippet, you may need to create migration to ensure its proper functionality. This is necessary as you may not have all the required columns in place.

```
# GET /books/ratings
def refresh_ratings
  # make a request to the api
  # get the ratings and the book title & author
  require 'open-uri'
  # the following url simulates a service call that returns 5 fake user objects
  url = 'https://random-data-api.com/api/users/random_user?size=5'
  json = get(url)
  json.each do |element|
    user = User.create(
      name: element['username'],
      email: element['email'],
      password: 'password',
      confirmed_at: Time.zone.now
    )

    book = Book.find_or_create_by(
      id: element['id'],
      title: element['address']['street_name'],
      content: element['address']['street_address'],
      user: user,
      publisher: user.name
    )

    book.rating = element['address']['coordinates']['lng'].to_i.abs
    book.save!
  end
  render json: Book.all
end

private

def get(url)
  JSON.load(URI.parse(url))
rescue
  raise 'Unknown'
end
```

## Timecop

[Timecop](https://github.com/travisjeffery/timecop) is a useful gem which is used for time manipulation to help you create specs.

Add a validation to the method `publish_if_live` in the Book model which ensures that no publications can be made during the weekends.

Create the necessary specs you seem fit to test this functionality.
