---
layout: post
title:      "My first paid coding opportunity, pt II"
date:       2020-09-12 22:04:08 -0400
permalink:  my_first_paid_coding_opportunity_pt_ii
---

To see the backstory, read [Part I](https://crankums.github.io/my_first_paid_coding_opportunity) first.

For my last post, I talked about the problem I was given to solve and how I would go about solving it. THEN! I posted a little of my code that dealt solving the core issue. Now we've got to build the rest!

We'll need to create some inputs: an area in which to paste the update and a submit button. Given that these would be the same every time, I figure I would just build these directly into the HTML of the app.

*(Is there a reason I shouldn't do this? Let me know in the comments!)*

Now that these are in place, let's get to the rest of the code.

I built the script out in class syntax, because it's just something I'm comfortable with. I could be convinced to rewrite the app in functional programming, but I was looking to get this done quickly, and I knew the framework for creating a vanilla JS app in this manner pretty well by now. 

With class syntax, I can put my bindings and event listeners right into my constructor, so let's start with that:

![App class with constructor, containing binding and event listeners](https://i.imgur.com/lGtnBo2.png)

Now I'll just cook most of this app's magic into the `handleSubmit()` function. We need a few things. First, we need to capture the input data. Then, we need a place to put it. 

Let's solve the first problem with:
```
let input = this.mainContainer.children[0].value
```
This is okay, because my page looks like all of:

![Screencap showing a textarea input and a submit button](https://i.imgur.com/gICKbhU.png)

And the text box will always be there, because it's hard-coded right into the HTML.

Next, we'll devote a few lines to creating a place to put the results:

```
  const resultsBox = document.createElement("div")
        resultsBox.setAttribute("id", "results-box")
        this.mainContainer.appendChild(resultsBox)
```

Alright, so we'll dump the results of the value extractor code I wrote in the previous post into this `resultsBox`.

![Screencap of page showing results as a comma-separated, unspaced string](https://i.imgur.com/cezvcaC.png)

Well, shoot. I suppose that since the comma is the delimiter, one could probably find a way to parse it out once they copy it to a spreadsheet. But the point of this is to *reduce* the amount of work as much as possible.  How should I organize the results? The people using this are going to drop it into a SQL tool that acts on the update IDs one row at a time. I think a table might be a good way of going about it.

So, we need to create a place for the table:

```
        let valueTable = document.createElement("table")
        valueTable.setAttribute("id", "value-table")
```

Then we need to create a series of rows, with one cell each, containing the update ID, the trick being to just make one long column of all the data.

```
let row = valueTable.insertRow()
        let perRow = 1
        let count = 0
        let resultsArr= this.valueExtractor(input)
        for (let i = 0; i<resultsArr.length; i++){
            let cell = row.insertCell()
            cell.innerHTML= resultsArr[i]
            count ++
            if (count%perRow == 0) {
                row = valueTable.insertRow()
            }
        }
        this.mainContainer.appendChild(valueTable)
```

let's see how that turns out:

![Screencap of page showing update IDs as a column](https://i.imgur.com/u18vPdi.png)

Much better. Now let's take a look at the `handleSubmit()`:

```
handleSubmit(e){
        const input =  this.mainContainer.children[0].value
        const resultsBox = document.createElement("div")
        resultsBox.setAttribute("id", "results-box")
        this.mainContainer.appendChild(resultsBox)        
        let valueTable = document.createElement("table")
        valueTable.setAttribute("id", "value-table")
        let row = valueTable.insertRow()
        let perRow = 1
        let count = 0
        let resultsArr= this.valueExtractor(input)
        for (let i = 0; i<resultsArr.length; i++){
            let cell = row.insertCell()
            cell.innerHTML= resultsArr[i]
            count ++
            if (count%perRow == 0) {
                row = valueTable.insertRow()
            }
        }
        this.mainContainer.appendChild(valueTable)
    }
```

I feel like it's gotten a little unweildy. In the future, let's look into refactoring this, maybe extracting the table creation functionality into its own function. That way, I could just call something like `this.tableCreator(this.valueExtractor(input))` and shorten up the length of the `handleSubmit()` function quite a bit.

Speaking of refactoring, I decided to clean up my `valueExtractor()` from the previous post. There were several considerations:
1) I basically forgot how `let` worked. So I kept declaring new variables everytime I had changed something with the input data, so a new variable to filter out the unwrapped characters, a new variable to turn the string into an array of strings, etc. I started to get worried about memory usage for something this small because I kept copying the data repeatedly to make a small change. That's why little projects like this are so valuable. They can remind of you how the basics work before you go running off with new frameworks and advanced tools.
2) Some issues were no longer a problem outside of a pure coding environment. Specifically, there was a problem with unwrapped/unescaped characters that I needed to remove in order to to the string into a series of iterable objects. The problem was the combination of double and single quotes made the update data string an invalid string unless I wrapped it in backticks. This was not an issue when it became input data, rather than a variable value I pasted into the text editor. I need to learn why it reads a string coming from the browser rather than the editor, but in the meantime, I was happy to not have to deal with that and begin manipulating the data directly.
3) Finally, I've been told by several people that I could likely have just solved this issue entirely with regex. That is possible, but I wanted to work with what I knew how to do rather than banging away at regex filters. 

Here is the refactored `valueExtractor()`:

```
    valueExtractor = (input) => {
        let stringBlob = input.split("u'").join("'").replace(/'/g, '"')
        stringBlob = stringBlob.split('},').map(e => e.endsWith(']') ? e+"}" : e).map(JSON.parse)
        let returnArr = []
        for (let el in stringBlob){
                returnArr.push(stringBlob[el]['__pk'][1])
            }
            return returnArr
        }
```

Thanks for reading!
