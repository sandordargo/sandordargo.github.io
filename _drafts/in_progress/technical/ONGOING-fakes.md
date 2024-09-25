Recently I had to touch some so called fakes. So what are suppposed to be fakes? Objects that can replace real implementations in tests. Objects that provide a very simplistic implementation that are good enough for tests.

Nevertheless, fakes should provide the same API as real objects.

I found something different. Something I didn't even comprehend first.

It simply had a very different API. A huge and different API.

Looking at the usage patterns didn't help a lot. It was even more confusing. Sometimes functions are called on it that are nowhere else present in the codebase. Sometimes a function with the same name that is in the api is called but there is no relationship between the API and the fake.

Again, sometimes a function called `client` is called that returns a real implementation! And all that is called a fake.

Speaking to the owners and looking into the signatures help a little bit more.

It's a mixture of a builder pattern and a whatnot.

Recognizing the builder pattern there is useful.