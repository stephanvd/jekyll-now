---
layout: post
title: Subatomic CSS Particles with Tailwind CSS
published: true
---

[Atomic Design](http://atomicdesign.bradfrost.com/) has seen some popularity with designers as a methodology to design UI systems. In the book it's described as follow:

> Atomic design is a methodology composed of five distinct stages working together to create interface design systems in a more deliberate and hierarchical manner.

It takes cues from nature and how bigger organisms are made up of molecules, which are made up of atoms. Organisms can then be composed into templates, which get instantiated into pages. 

Most apps probably follow at least some form of this pattern implicitly or explicitly. They have modular components which get composed into pages. And those components might be composed of multiple levels of sub-components. The underlying pattern is not new and has some well known values: Modularity and composability. What Atomic Design adds to the conversation is more nuance and a shared language to talk about these distinct levels.

## CSS belongs in the component

> Design components that are self-contained, independent, and have a single, well-defined purpose.
>
> -- **The Pragmatic Programmer**

For modularity we need low coupling and high cohesion. Low coupling helps us isolate the component from externalities for ease of reuse. High internal cohesion is about how the pieces of a component fit together.  

SPA frameworks like React and Vue brought forth a big change in thinking: HTML and JS mixed together allow for high cohesion and low coupling. CSS was initially left as the odd one out. When building a cohesive React component we still need to navigate a seperate tree of CSS files to write some styles to target our component. And we should keep in mind what else the style we're touching is impacting while we're at it. We might just break something. As our app grows this risk increases.

The culprit here is global state. Global CSS is bad. It's difficult to comprehend what influences what, inflexible to overwrite, and more complex selectors can impact performance. Who hasn't been scared to touch a piece of CSS because they were unsure what else would change? Add in te power of SASS and complexity gets even worse. [BEM](http://getbem.com/) can help us minimize some of the risks involved. Reusing CSS becomes more deliberate. But reusing CSS still means duplicating the HTML structure. And we're still navigating a seperate tree of CSS files. Where's the cohesion in that? The coupling here hints at a solution we saw with HTML and JS before: Pull CSS into the component!

With CSS in the component all we need is simple flat CSS seletors. We alway know the scope our changes apply to. It's all self contained. Now that's low coupling, high cohesion! I know, I know. It's possible to write flat selectors with BEM. It's possible to scope selectors to components. It's possible to use SASS for scoping. But while *possible*, it's not the path of least resistance. It takes effort. It's tempting to break the rules, take a shortcut. CSS in components gives us a sane default spot to put our stuff. Predictable and visible. Tools like CSS-in-JS offer this solution.

## 

---
TODO:
- High cohesion, low coupling
- CSS is quantum: fuzzy, needs to be entangled to use
- Tailwind collapses the wave function into a sane reality
- Noisy? Go smaller!
