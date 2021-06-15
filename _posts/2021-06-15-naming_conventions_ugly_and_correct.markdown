---
layout: post
title:      "Naming Conventions, ugly and correct?"
date:       2021-06-15 15:01:29 +0000
permalink:  naming_conventions_ugly_and_correct
---


I just finished building my first single page web application using Javascript. I fairly quickly ran into an issue that i had never considered before. I built an API for the backend data using Rails, simple enough. I was making fetch calls to my API and getting the proper responses. Everything was going great. And that’s when I had to instantiate a new Javascript class object.

One of my models in rails was named Restaurants. It had an attribute called top_dishes. So when building out the constructor for my Javascript object I typed:

`this.topDishes = top_dishes`

And honestly, it felt wrong.

I hadn’t considered the fact that I’d need to break naming conventions in one of my languages to get everything to work properly. Should I even be changing the name of the attribute in one of my languages?

Naturally I looked to the internet for a solution, but didn’t find nearly as much information as I’d expected. I expected to find a set-in-stone hard stance on the matter from the community, but the few articles and reddit posts I found about it were mainly just debating the merits of either system.

The one consistent agreement people had was that naming convention should be upheld in each language. The discrepancy was where the name change should take place. I seemed to find more people in support of changing the name on the Javascript side of things, and leaving the data on the API alone. This way you could rename the data in the fetch response or in the constructor, thus making the API easier and more predictable to communicate with. However the arguments for the other side were strong as well. One big point being that it’s bad practice to expose your server data to a client. If your server key names are being pulled directly from your API, this exposes them to the client, which can leave you more vulnerable to attacks. To prevent this you’d likely want to rename your data before pushing it to the API anyway, in which case you could rename it for Javascript at that point.

I ended up going with the former method. It’s a very small app, and will not be out in a production environment, so I’m not worried about exposing data. This route kept everything relatively simple as well, as the only places I had to implement the data change were in the constructor, and when submitting the data back to the API for a post/patch request. In the future however I will most likely go the route of protecting my data and changing it on the API. The importance of data security seems to be getting more and more crucial every day, and the idea of hiding as much of the backend data from the front seems to me like a much smarter direction for the future.

That all being said, if you are set on keeping the naming conventions proper in both locations, and you have more attributes to process than I did, there are cool gems like OliveBranch (https://github.com/vigetlabs/olive_branch) that can do the work for you. Olive Branch is rails 
middleware that converts camel and dash cased inputs to your API to snake case, all just by adding an extra header to your request (X-Key-Inflection). Now everyone gets the data they want! What a time to be alive!

It’s always funny the rabbit holes that a seemingly simple line of code can take you down. Especially when you think it’s an issue that HAS to have an agreed upon solution. It’s a nice reminder that we’re all learning and growing, and that often times there’s no one size fits all solution to a problem. We just have to make the best decision we can with the information we can find, and learn all we can from the experience.
