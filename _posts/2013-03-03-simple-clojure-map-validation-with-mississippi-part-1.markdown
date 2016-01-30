---
layout: post
title: Simple clojure map validation with Mississippi (part 1)
---
#Simple clojure map validation with Mississippi (part 1)
### The problem ###


We have a product at work which needs to send data to a number of different enterprise-level companies (UK energy suppliers). Some of these companies need that data to be in a csv format,
some need it to be in an xml format, which for the most part is not a problem. What becomes difficult to manage is the limitations on what types of values can
be put in each one of the fields we send across.

Traditionally, I suppose, this kind of thing would have been managed using strict schemas and change documents, but none of this is really conducive to the rapid
development cycle we are used to. We had already re-written most of the product in clojure, turning a monolithic C# application into a number of much smaller, and 
easier to manage components (you can see my colleague Ryan Greenhall speaking about this transisition in this video:)

As such we already had our data represented in Clojure maps, so, for example, if an energy supplier had a spec which looked a bit like this:

Field            Value
First Name       Customer's first name (max 20 chars)
Surname          Customer's last name (max 20 chars)
Email Address    Customer's email address (max 50 chars, must be valid email address)

We would translate the customer information, stored as json, as follows:

` {:FIRST_NAME  (customer-information :first-name)`
`  :SURNAME     (customer-information :surnname)`
`  :EMAIL       (customer-information :email)}`

That wouldn't be too bad, except that different suppliers had wildly different restrictions on what could be put in each field, for example, here are two pictures
of 

As you can see, the formatting of the values are different, the range of values allowed are different, and the occasions when this field is mandatory are different.  
Now as most people know, the first rule of web development is don't trust the client, and we did have a number of different validations in place on the front end
to prevent invalid input, but with over 20 different suppliers requiring between 30 and 200 pieces of individual customer information, each of which has it's own
data format requirements, it swiftly become unmanageable. The only way to ensure correctness is to manage the supplier requirements at what we refer to as the 
"translation" phase, when we modify the customer information into the format the supplier needs.

Now it's tempting to encode these requirements in large branching logic operations, or complex strategy patterns but this quickly become unmanagable. What we wanted
was something which checked whether our naive translations would allow the supplier to process the file, and warn us on the occasions that the details the customer
had entered could not be moulded into the supplier specific format.


### Enter Mississippi ###

Mississippi was a small clojure validation library written by Mike and Gareth Jones. It was written on a plane to the Strange Loop conference in the US, and the first 
commit was made over Mississippi, hence the name.

Here is an example of Mississippi in action

`(use '[mississippi.core])`
`(errors {:a nil :b \"value\"}`
`       {:a [(required)] `
`        :b [(required)]}`
  
`=> {:errors {:a (\"required\")}`

You bring in the namespace, and then you can validate that a subject matches some constraints that you have applied. The errors function is a mississippi core function
which takes as it's first argument a map of values to validate. It's second argument is then a map of predicates the subject must satisfy to be considered valid.

In this case both :a and :b are defined as 'required', so when a map with no value for :a is passed in, a collection of errors, with a map containing :a and a message is
returned

(more to follow)