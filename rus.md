JavaScript has come a long way since its early versions and thanks to all
efforts done by[TC39][1] (The organization in charge of standardizing
JavaScript (or**ECMAScript** to be exact)) we now have a modern language that
is used widely.

One area within **ECMAScript** that received vast improvements is **
asynchronous code**. You can learn more about
[asynchronous programming here][2] if you’re a new developer. Fortunately we
’ve included these changes in Windows 10’s new Edge browser. Check out the
change log below:

[https://dev.modern.ie/platform/changelog/desktop/10547/?compareWith=10532][3]

Among all these new features, let’s specifically focus on “ES2016 **Async
Functions**”** **behind the **Experimental Javascript** features flag and take
a journey through the updates and see how ECMAScript can improve your currently
workflow.

## First stop: ECMAScript 5 – Callbacks city

**ECMAScript 5** (and previous versions as well) are all about callbacks. To
better picture this, let’s have a simple example that you certainly use more
than once a day: executing a XHR request.

    var displayDiv = document.getElementById("displayDiv");

    // Part 1 - Defining what do we want to do with the result
    var processJSON = function (json) {
        var result = JSON.parse(json);

        result.collection.forEach(function(card) {
            var div = document.createElement("div");
            div.innerHTML = card.name + " cost is " + card.price;

            displayDiv.appendChild(div);
        });
    }

    // Part 2 - Providing a function to display errors
    var displayError = function(error) {
        displayDiv.innerHTML = error;
    }

    // Part 3 - Creating and setting up the XHR object
    var xhr = new XMLHttpRequest();

    xhr.open('GET', "cards.json");

    // Part 4 - Defining callbacks that XHR object will call for us
    xhr.onload = function(){
        if (xhr.status === 200) {
            processJSON(xhr.response);
        }
    }

    xhr.onerror = function() {
        displayError("Unable to load RSS");
    }

    // Part 5 - Starting the process
    xhr.send();

Established JavaScript developers will note how familiar this looks since XHR
callbacks are used all the time! It’s simple and fairly straight forward: the
developer creates an XHR request and then provides the callback for the
specified XHR object.

In contrast, callback complexity comes from the execution order which is not
linear due to the inner nature of asynchronous code:

![image][4]

The “[callbacks hell][5]” can even be worse when using another asynchronous
call inside of your own callback.

## Second stop: ECMAScript 6 – Promises city

**ECMAScript 6** is gaining momentum and Edge is [has leading support][6] with
88% coverage so far.

Among a lot of great improvements, **ECMAScript 6** standardizes the usage of*
promises* (formerly known as futures).

According to [MDN][7], a *promise* is an object which is used for deferred and
asynchronous computations. A*promise* represents an operation that hasn’t
completed yet, but is expected in the future. Promises are a way of organizing
asynchronous operations in such a way that they appear synchronous. Exactly what
we need for our XHR example.

> *Promises* have been around for a while but the good news is that now you don
> ’t need any library anymore as they are provided by the browser.
>

Let’s update our example a bit to support *promises* and see how it could
improve the readability and maintainability of our code:

    var displayDiv = document.getElementById("displayDiv");

    // Part 1 - Create a function that returns a promise
    function getJsonAsync(url) {
        // Promises require two functions: one for success, one for failure
        return new Promise(function (resolve, reject) {
            var xhr = new XMLHttpRequest();

            xhr.open('GET', url);

            xhr.onload = () => {
                if (xhr.status === 200) {
                    // We can resolve the promise
                    resolve(xhr.response);
                } else {
                    // It's a failure, so let's reject the promise
                    reject("Unable to load RSS");
                }
            }

            xhr.onerror = () => {
                // It's a failure, so let's reject the promise
                reject("Unable to load RSS");
            };

            xhr.send();
        });
    }

    // Part 2 - The function returns a promise
    // so we can chain with a .then and a .catch
    getJsonAsync("cards.json").then(json => {
        var result = JSON.parse(json);

        result.collection.forEach(card => {
            var div = document.createElement("div");
            div.innerHTML = `${card.name} cost is ${card.price}`;

            displayDiv.appendChild(div);
        });
    }).catch(error => {
        displayDiv.innerHTML = error;
    });

You may have noticed a lot of improvements here. Let’s have a closer look.

### Creating the promise

In order to “promisify” (sorry but I’m French so I’m allowed to invent
new words) the old XHR object, you need to create a*Promise* object:

![image][8]

### Using the promise

Once created, the *promise* can be used to chain asynchronous calls in a more
elegant way:

![image][9]

So now we have (from the user standpoint):

*   Get the promise **(1)**
*   Chain with the success code **(2 and 3)**
*   Chain with the error code **(4)** like in a try/catch block

What’s interesting is that chaining *promises* are easily called using *.then
().then
(),* etc.

> **Side node:** *Since JavaScript is a modern language, you may notice that I
> ’ve also used
>*[*syntax sugar*][10]* from **ECMAScript 6** like *[*template strings*][11]*
> or
>*[*arrow functions*][12]*.*

## Terminus: ECMAScript 7 – Asynchronous city

Finally, we’ve reached our destination! We are almost in the [future][13], but
thanks to Edge’s rapid development cycle, the team is able to introduce a bit of
**ECMAScript 7** with **async functions** in the latest build!

Async functions are a syntax sugar to improve the language-level model for
writing asynchronous code.

> Async functions are built on top of ECMAScript 6 features like generators.
> Indeed, generators can be used jointly with promises to produce the same results
> but with much more user code
>

We do not need to change the function which generates the promise as async
functions work directly with promise.

We only need to change the calling function:

    // Let's create an async anonymous function
    (async function() {
        try {
            // Just have to await the promise!
            var json = await getJsonAsync("cards.json");
            var result = JSON.parse(json);

            result.collection.forEach(card => {
                var div = document.createElement("div");
                div.innerHTML = `${card.name} cost is ${card.price}`;

                displayDiv.appendChild(div);
            });
        } catch (e) {
            displayDiv.innerHTML = e;
        }
    })();

This is where magic happens. This code looks like a regular synchronous code
with a perfectly linear execution path:

![image][14]

Quite impressive, right?

And the good news is that you can even use async functions with arrow functions
or class methods.

##

## Going further

If you want more detail on how we implemented it in Chakra, please check the
official post on Edge blog:
[http://blogs.windows.com/msedgedev/2015/09/30/asynchronous-code-gets-easier-with-es2016-async-function-support-in-chakra-and-microsoft-edge/][15]

You can also track the progress of various browsers implementation of **
ECMAScript 6** and **7** using [Kangax’s website][6]: Feel free also to check
[our JavaScript roadmap][16] as well!

Please, do not hesitate to give us your feedback and support your favorite
features by using the vote button:

![image][17]

Thanks for reading and we’re eager to hear your feedback and ideas!

**David Catuhe**

Principal Program Manager

[@deltakosh][18]

 [1]: http://www.ecma-international.org/memento/TC39-M.htm
 [2]: https://msdn.microsoft.com/en-us/library/hh191443.aspx

 [3]: https://dev.modern.ie/platform/changelog/desktop/10547/?compareWith=10532 "https://dev.modern.ie/platform/changelog/desktop/10547/?compareWith=10532"
 [4]: img/8816.image_5F00_thumb_5F00_35C48EA5.png "image"
 [5]: http://callbackhell.com/
 [6]: http://kangax.github.io/compat-table/es6/

 [7]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
 [8]: img/6305.image_5F00_thumb_5F00_56493DA6.png "image"
 [9]: img/6012.image_5F00_thumb_5F00_0D8FC4E2.png "image"
 [10]: https://en.wikipedia.org/wiki/Syntactic_sugar
 [11]: http://tc39wiki.calculist.org/es6/template-strings/
 [12]: http://tc39wiki.calculist.org/es6/arrow-functions/
 [13]: http://kangax.github.io/compat-table/es7/
 [14]: img/1665.image_5F00_thumb_5F00_0BBF6F1B.png "image"

 [15]: http://blogs.windows.com/msedgedev/2015/09/30/asynchronous-code-gets-easier-with-es2016-async-function-support-in-chakra-and-microsoft-edge/ "http://blogs.windows.com/msedgedev/2015/09/30/asynchronous-code-gets-easier-with-es2016-async-function-support-in-chakra-and-microsoft-edge/"
 [16]: https://dev.modern.ie/platform/status/javascript/
 [17]: img/1348.image_5F00_thumb_5F00_49FA5F60.png "image"
 [18]: http://www.twitter.com/deltakosh
