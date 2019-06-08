# Foreword

The purpose of this book is to give me an introduction to advanced topics and to convey best practices on React. If you not only understand how something works with the book, but also why, then I have achieved my goal. Now every developer has different ideas about which methods are the best and how to write the simplest, most efficient or most beautiful code. However, I strongly adhere to the recommendations of the core developers on Facebook, the recommendations of AirBnB, which were also well received by the community, and some other greats from the "React scene". All seasoned with a pinch of personal experience.

For example, there are several ways to publish your application later, whether you bundle it with tools like **Browserify, Rollup** or **Webpack** or not. Whether you write your components as ES2015 classes or use `createClass` from "ES5 times". Where I think it makes sense, I will go into the various common methods in order not only to give directions, but also to show alternatives.

However, I would like to primarily deal here with the most modern, up-to-date and in most cases also simplest methods, which is why I will assume for most code examples a setup with **Webpack**, **Babel** and **ES2015** \(and newer\), which I will describe very exactly again in the further process. Anyone who has never been in contact with ES2015+ before will certainly need a moment longer to understand the examples, but I will try to keep all the examples understandable and also go into ES2015+ in more detail. However, JavaScript basic knowledge should be available at the time of reading.

This book also only covers the **Entry to React** topic and does not provide an introduction to JavaScript. Basic and in a few places certainly some deeper knowledge of JavaScript is required, whereby I explain everything in a beginner-friendly way, even if one was only superficially in contact with JavaScript so far. I don't assume that every reader can explain error-free how a JavaScript interpreter works, but I do assume that the reader is reasonably familiar with how scopes work in JavaScript, what a callback is, how `Promise.then()` and `Promise.catch()` work, and how the principle of asynchronous programming works with JavaScript.

\*\*But don't worry, it sounds more complicated than it actually is in the end. Any reader who has worked with jQuery in the past should have no problems understanding the majority of this book and should be able to follow my explanations.

## A few words about this book

I have published this book in **Selbstverlag** because I want to keep full control over all publishing channels, the pricing model, and all rights and freedoms over the book. I'm not interested in making "the big riches", I want to make this book accessible to as many people as possible. For this purpose there is a free online version of this book, which you can find at the URL [https://lernen.react-js.dev/](https://lernen.react-js.dev/). With the purchase of the ePub or PDF version via iBooks, Google Play Books, Amazon Kindle or Leanpub you can financially support my work on the book and get a nice e-book for download instead of the HTML version.

Since **Selbstverlag** also includes the term "self", I was completely on my own. No publisher who has provided me with an editor or his distribution infrastructure. However, a proofreader has already been assigned \(as of the end of April\), but this will take a while until he is finished with the proofreading. For this reason this book is perhaps not as round at some corners as one is used to from other specialist books with a publisher in the back. I beg your pardon for that. If you should find an error, no matter if content, grammar or just a typo, you can always create a [Ticket on GitHub](https://github.com/manuelbieh/react-book/issues) for me.

The book is written completely in Markdown format and is also available [completely public on GitHub](https://github.com/manuelbieh/react-book). For writing I used [Gitbook.com](https://www.gitbook.com/). Basically, I loved it, hated it a few days. The service is fantastic for writing technical documentation, less actually for writing whole books, even if the name suggests it.

If you like the book or have a question, the best way to reach me is via Twitter. Here I am glad about every feedback, no matter if encouraging, positive words or constructive criticism!

And now I wish you a lot of fun with the book!

