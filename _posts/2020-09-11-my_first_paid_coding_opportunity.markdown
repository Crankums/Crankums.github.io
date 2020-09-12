---
layout: post
title:      "My first paid coding opportunity"
date:       2020-09-11 23:05:08 -0400
permalink:  my_first_paid_coding_opportunity
---

Hopefully, this will be the start of something good.
Currently, I'm employed in the operations division of market data company. I won't go into all that we do there, but I will say that 1) it's a tech company, and 2) we maintain a huge database. One of our biggest pools of employees are the people that basically ensure that the data we maintain is properly "bucketed" for use by our client-facing products. Any updates made today will be shown in our product's tables the next day, which gives us a chance to check and fix any mistaken attributions that may occur. Our attributions are done either the people mentioned above or by more than a few automated systems we have. If I ever get a chance to work with said systems directly, I'll share what I've learned, as long as it doesn't violate any sort of internal confidentiality.

This job doesn't really give me much opportunity for coding. There's a little exposure to SQL, but my position mostly involves supervising and training users of our internal database client. However, my manager presented me with an opportunity when he observed a particular issue: when a user makes a big update, our logging service provides a large string with the object ID numbers of the updated database rows throughout. He said that that if there was a way to extract the IDs, it would save a lot of time. This is not a big deal if it were only a dozen or so values to extract, but the only time these ID numbers need to be pulled is when an update is erroneously made in the thousands.

Taking a look at these strings, I noticed that they were effectively strings of JSON objects (note: I've always wondered if that's redundant? I could also say it's a string of objects, but adding "JSON" does immediately tell you what shape the object comes in.) This would be something iterable if you converted the data type from a big string to a series of objects, like an array or a bigger object (footnote: or hash, for the Rubyists, dictionary for the Python-types, etc), and a simple matter of asking for the value to the reoccuring key.

![A screencap of update data string](https://i.imgur.com/CRWlBlY.png)

We can see the object ID, underlined above, has the same key. We can also see the problem: as of right now, the whole "blob" of update data is fine as a string, but runs into problems when we pass it into our code. Mainly, there are unwrapped characters in every set of curly brackets, preventing the code from being read as a string or as a series of objects if we were to convert it by means of something like `JSON.parse()`. 

Now what I believe I would like to do is return a simple array of values, presented as either strings or integers (Should one be favored by another in the use case? Tell me why below!). Maybe this problem could be solved with some regex, but I'll be honest, I'm not great at it, and I haven't tested the variability of our update data enough to be sure that I'd be able to apply a consistent filter that would apply to every data pool passed in.

Furthermore, I'd like to be able to work with this data with as little manipulation before passing it in as possible. With the weird little unwrapped/unescaped characters in every set of curly brackets, this means I'll have to take some extra steps.

To start off, I copied the update data to a document editor and wrapped it in backticks. This action turned it to a literal string, so the unwrapped/unescaped characters would cause any issue. Now I can freely use a little regex to eliminate those characters. While I'm at it, why don't I turn all those single quotes into double quotes? This will be important later.


`.replace(/'/g, '"')
`


Next, I'll `split()` the string into an array of strings, then loop through it add the curly bracket and comma I chopped off back in. (Is there a better way of doing this? Let me know in the comments!) Now I can iterate through the array and parse each of those strings into a JSON object. If I had left the single quotes in each object, I would not be able to use this function, it's very particular about that, as I discovered!

Finally, I can now iterate the objects for the key-value pair I need, and extract the values I need to make the update far easier. 


```
const  valueExtractor = (input) => {
  let stringBlob = input.split("u'").join("'").replace(/'/g, '"')
  let arr1 = stringBlob.split('},')
  let arr2 = arr1.map(e => e.endsWith(']') ? e+"}" : e)
  let arrParse = arr2.map(JSON.parse)
  let values = []
  for (let el in arrParse){
  values.push(arrParse[el]['__pk'][1])}
  arr2=[]
  arr1=[]
  return values
}
console.log(valueExtractor(input))```


I cleared out the arrays before the return to ensure that this code would minimize memory bloat. It might not be important or even the correct way of doing it, but I figured it was a precaution that wouldn't do any harm to take. 

Some refactors would include learning more about manipulating/mutating objects in place so that I don't need to declare as many variables, and seeing how many of these object changes I can compress to single lines. 

For extra fun, feel free to point a fundamental attribute of *let* keyword that I seem to have neglected!

Next step, building the page. Let's see if I can handle the code anxiety!
