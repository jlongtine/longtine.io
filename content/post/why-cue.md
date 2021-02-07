+++
date = 2021-01-25T07:00:00Z
draft = true
layout = ""
title = "Why I chose CUE"

+++
A few pieces of context before I dive in. This is intended to be the first of many posts about CUE and how me and my team at Gloo (gloo.us) are using it on our Site Reliability/Platform Team. If I'm not publishing more posts... I give you permission to give me shit about it (nicely, please) on [Twitter](https://twitter.com/jlongtine). Along those lines, if there are aspects of CUE that you'd love to know more about, feel free to let me know, on Twitter or CUE Slack! I'd love your feedback, as well. 

Onward!

There are a lot of config languages and tools out there. 

YAML, JSON, HCL, Jsonnet, Kustomize, TOML... and so many more.

Me and my team at Gloo (gloo.us) were using YAML CloudFormation templates with nested stacks and got to a place where we needed something better, something more. 

I found [Jsonnet](https://jsonnet.org/), and started to use it. There's a lot to like about it, honestly. I think the biggest benefits of it are around boilerplate reduction and some ability to standardize the structure of your configuration. Kubernetes labels come to mind. But something about the function -> struct and override mechanisms didn't quite sit well with me. So, we kept looking. 

We came across CUE in April/May of 2019. Had the opportunity to connect with Marcel, and got a sense of where he was planning on taking the language.

Our rationale at the time for choosing it was not very well defined. I think an honest assessment would be that we intuitively believed that it would be a better fit for our use cases in the long run. Thankfully, I've seen that born out over the last 20 months or so of use. 

So, I think it'd be useful to speak to the features of the language that had us choose it, and then give a sense and examples of how we're currently using in our infrastructure at Gloo. 

As our infrastructure grew in complexity, and given where I imagined it would continue to go... I knew we had to make sure that we had tools to keep that complexity in check.

CUE is a relatively simple language... But a few of its feature combine to create a lot of power. Commutativity + Unification. Value Lattice/Constraints. 

Let's start with **Commutativity**. From [Wikipedia](https://en.wikipedia.org/wiki/Commutative_property):

> In [mathematics](https://en.wikipedia.org/wiki/Mathematics "Mathematics"), a [binary operation](https://en.wikipedia.org/wiki/Binary_operation "Binary operation") is **commutative** if changing the order of the [operands](https://en.wikipedia.org/wiki/Operand "Operand") does not change the result.

As described in depth in that article, this basically means: order doesn't matter. In CUE there is no overriding of values. If a value is set to something concrete, it can't be another concrete value somewhere else.

Unification describes what happens with a given field across files. 

As an example:

    // one.cue
    fruit: "apple"
    
    // two.cue
    fruit: "banana"

This will not evaluate in CUE, because when the fields are unified, they are not equal to one another.

As a potentially interesting counter-example, this _does_ evaluate in CUE: 

    // one.cue
    fruit: "apple"
    
    // two.cue
    fruit: string

This is where the Value Lattice and Constraints come into the picture. In `two.cue`, `fruit: string` defines a constraint that `fruit` be of type `string`. That gets unified with `"apple"`, which... surprise... is a `string`. This applies for all of the types in CUE. 

    i: int
    i: 10
    
    // which is equivalent to 
    i: int & 10
    
    // & is the unification operator that is also implied by defining a field multiple times. 

There are a number of other features in the language that help to flexibly apply these ideas against configuration, but these are the really important features, in my opinion. They provide the backbone on which everything else sits.

From here, I think it's worthwhile to show a few examples of how we're use CUE at Gloo. I think it might help clarify why I believe these features are so important. 

### Why do we need another config language?

There is YAML, JSON, HCL, Jsonnet, TOML, and many more.  
What’s the point of adding another one?

Also, why not simply use a Turing complete language like Python or Ruby to express these things?

That is part of what I’m wanting to explore in this post.  
I’ll speak to the weakness of the above approaches.