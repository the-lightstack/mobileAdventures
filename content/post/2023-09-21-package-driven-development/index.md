Everyone who has worked on a bigger project in flutter knows that the standard library with its widgets will only get you so far. If you are lucky someone has already fixed your problem on [pub.dev](https://pub.dev) or [stackoverflow](https://stackoverflow.com). If not, this article is for you as we will explore how to:
1) Fix your problem easier, more quickly and reliable
2) Help out the community
3) Increase your chances of getting a job

## Put it in a box!
If your application requires a fairly complex UI (or backend logic) component, putting it into its own package has several advantages. For one, you can work on it in total isolation. Had you directly implanted it in your final application, bugs from there would have thrown you off track and slowed down the process. This modularization also keeps your tests clearly separated. 
## Sharing is caring
The package you write is also not just for you, but _can_ be shared with the entire flutter community, saving your companions **countless** hours. This in turn might make you feel pressured to write clean and maintainable code. Which is not too bad in my opinion, as it will save you time in the long run.
Assuming you decided to publish your package on [pub.dev](https://pub.dev) you will also get benefits out of it:
1) People contributing and improving your component
2) Making a good impression on your portfolio

## Not just for others
You yourself will also benefit from stuffing all that code into a fresh package. It forces you to generalize the problem which will also have you thinking about thinks that could go wrong and you should therefore write tests for. 
In addition you can reuse it with ease in all your future projects, just by adding it to your dependencies. This also provides consistency for your users who have learned how to properly use this one part of your application. They can transfer this knowledge and have a better experience, leaving you better reviews. 

## What to consider
Leave the styling and some implementation choices mostly up to the developer who is going to integrate it. 
A package that let's you create charts should leave colors and ways of interacting up to the one implementing it.

## Want more?
If you like this type of short article and want to explore more flutter concepts coming in the future , consider subscribing to my [→ My Newsletter 🗞️  ←](https://buttondown.email/Lightstack?tag=article)😁.
