---
title: "SKI-ing into the Technical Interview"
tags: [Aphyr, Interview, Combinators, Julia]
date: 2023-09-25
---

# SKI-ing into the Technical Interview







By Mark Dixon
#### _____

This note is more a homage to the excellent series of of [blog entries by Aphyr](https://aphyr.com/tags/interviews) parodying coding interviews. The original is leagues beyond anything I could ever hope to accomplish, but after a number of coding sessions trying to get to the bottom of some combinator programming in Julia I could't resist the urge to use it here.
## SKIing on the Technical Interview

You trail behind Tim into the brightly illuminated chamber. A tad too radiant for your preferences, you sense a wave of apprehensive dread beginning to surge. Perhaps the whispers you've heard about this small tribe hold truth - your instincts urge you to retreat, but Tim is swifter and ushers you towards a chair.

"Ok, so, you've probably done this a thousand times before, judging by your beard," he begins, "so lets just get right to it."

Not sure if to take him seriously or not, your focus drifts up from Tim's feet and stops at his neckbeard. Tim won't journey far in this realm. But, regrettably, you need his gold.

Pointing to the sleek, modern teletype before you, Tim continues: "Right, so lets start with something easy to get you warmed up. How about a function that returns the factorial of a number?"

You nod, feeling a sense of relief as things start to drift into your familiar fjords. You were weaned on factorials as a whelp back in the northern homelands.

"What runes does your clan inscribe?" you inquire, recalling cousin Aisha's sagely counsel when adjusting to the earthbound ways of working.

"Runes? Oh, you mean programming languages? We're primarily a Go company but feel free to use whatever you want really. But do try to be practical - nothing fancy, I just want to see how you'd work day to day problems."

Your aprehension is now replaced by confusion. Tim clearly understands that functions are the source of existence. So how could they be considered fancy? Oh&hellip; you get him now. So how do mortals code now again..?

Your fingers dance over the keys --

"We have Emacs on this machine?!"

&mdash; and summon forth a file from the depths of Helheim.

```julia
using SpecialFunctions
ᚠ = factorial
ᚠ(10)
```



"Wait, .jl? What language is that? Julia?" Tim's brow furrows in confusion.

"You requested practical," you respond with a calm patience, "so I chose a syntax as grounded as Midgard itself. We're not trying to re-forge Mjölnir here, are we?! Or perhaps you'd have preferred if I'd navigated the tides of Njord's realm?"

"Riiight.. but, you know, I was kind of hoping you would show how to build up a factorial function yourself and not just use a built-in functon," Tim responds. "Ok, let's try again. How about you write a function to compute the nth Fibonacci number. This time just stick to basic native language features, you know? Show how you would build up the function from scratch."

The scent of the mortal before you grows increasingly potent, a stark reminder of your differences.

"So, we're to abandon the ancient scrolls of knowledge? Construct it from the bare bones of the elements?" you inquire.

Tim nods, "Sure, yeah the elements, whatever. You can stick with Julia if you want."

A sense of alignment washes over you, as if you and Tim are sailing the same celestial river. Could you have misjudged him? Unlikely, but you are well-versed in this kind of task. The function operator is as familiar to you as the back of your hand. You could perform it in the depths of Odin's sleep. Clearing the buffer, you close your eyes and summon those most ancient of runes:

```julia
const ᛝ = ᚷ->ᚷ
const ᚴ = ᚷ->ᛉ->ᚷ
const ᛣ = ᚠ->ᚷ->ᛉ->ᚠ(ᛉ)(ᚷ)
const ᛒ = ᚠ->ᚵ->ᚷ->ᚠ(ᚵ(ᚷ))
```

"Woah! Hold on there," Tim interjects, he voice echoing in the chamber. "I've seen this sort of trick before. Stick to the Roman alphabet please."

*Heathen!* You briefly contemplate leaving the chamber, but then remember that you need this more than Tim needs you. Curse this age of ignorance.

Your fingers dance over the keys like the Norns weaving the threads of fate, transmuting your incantation into the language of the unenlightened. A silent invocation escapes your lips &mdash; may the potency of the runes remain undiminished in this foreign form. You've never dared to test their might in such a crude manner before:

```julia
const I = x -> x
const K = x -> y -> x
const C = f -> x -> y -> f(y)(x)
const B = f -> g -> x -> f(g(x))
```

"Alright, that's more like it," Tim exhales, a hint of relief in his voice. "For a moment, I thought you were one of those eccentric types we encounter occasionally. But what's the connection to the Fibonacci sequence here?"

"Patience, Tim, as steady as Yggdrasil's roots." You attempt to soothe him with your Voice, resonating with the timbre of ancient sagas. "We will reach that fjord soon. But first, we must invoke the aid of the primal elements."

Your fingers speed up, not wanting Tim's skepticism to disrupt the Seidr taking hold of you.

"First the opposing forces of Fire and Ice. Muspelheim and Niflheim. Or the duality of right and wrong as you would say in your tongue."

```julia
const T = K
const F = K(I)
```

"Oh, true and false! Ok but try to name your variables a bit more descriptively," Tim is definitely looking a tad worried now. "And explain what you are doing as you go along, I need to understand how you think." 

You don't wish to unsettle him, so you promptly adjust to his requests. Not that Tim seems to comprehend the difference between mere variables and the fundamental elements of the cosmic abyss. The art you're practicing isn't 'thinking' in the conventional sense - it's more akin to allowing Yggdrasil to channel its wisdom through you.

"I don't exactly think, I&hellip; 'thunk' in your parlance, perhaps? You see, things are always most potent in twos," you attempt to elucidate your methods. "I believe you'd refer to them as pairs?"

```julia
const pair = B(C)(C(I))
const first = C(I)(T)
const second = C(I)(F)
const next = n -> pair(F)(n)
const prev = n -> second(n)
```

"Ah, ok, so you are building a tuple class? I thought we were going to start with something simple."

Tim's words barely register. The ancient incantations are beginning to dominate your consciousness. Your runic flow is potent. What comes next? Numbers! Naturally, you require the essence of the Norns:

```julia
const 𝟘 = I
const 𝟙 = next(𝟘)
const 𝟚 = next(𝟙)
const 𝟛 = next(𝟚)
const 𝟜 = next(𝟛)
const 𝟝 = next(𝟜)
const 𝟞 = next(𝟝)
const 𝟟 = next(𝟞)
const 𝟠 = next(𝟟)
const 𝟡 = next(𝟠)
```

"Alright, that's a bit unconventional, but I think I see where this is heading," Tim tries to convince you he is still in control here: "But Julia doesn't have lazy evaluation right?"

*Foolish mortal. Do you think you can outwit me?*

One final surge of energy, and now you only need to thunk the final runes for this incantation to be complete:

```julia
const isZero = n -> first(n)
const ifThenElse = I
const apply = n -> f -> x -> () -> 
    ifThenElse(isZero(n))(()->x)(()->f(apply(prev(n))(f)(x)()))()
const add = m -> n -> apply(m)(next)(n)()
```

Satisfied, you recline in your chair. That should suffice. You must truly be appeasing Tim now &mdash; pure functions extending all the way down to Yggdrasil's roots.

Tim tilts his head, scrutinizing the screen. A lengthy silence follows.

"Tim..? The Fibonacci sequence, yes? Even these hallowed numbers adhere to the ancient laws of duality," you attempt to reassure him. "All we need to do now is to summon the correct pair&hellip;"

Tim nods. That's your cue. Time to cast the spell:

```julia
const nextFibPair = p -> pair(second(p))(add(first(p))(second(p)))
const fib = n -> first(apply(n)(nextFibPair)(pair(𝟘)(𝟙))())
```

Your flow ebbs away. The tension in the room begins to dissipate.

"Ok. Great. Um," he mumbles. "So how do we know if this works? Can you, like, write some unit tests or something?"

*Foolish!* Of course. Once again, you've overlooked Aisha's counsel. These mortals never place their faith in the gods. They demand their own proofs.

"Certainly," you respond, attempting to placate him. "But then I'd have to invoke more than just functions&hellip; perhaps integers and the addition rune would work, hope that's acceptable?"

```julia
const toInt = n -> () ->
    ifThenElse(isZero(n))(() -> 0)(() -> 1 + (toInt(prev(n)))())()

@test toInt( fib(𝟘) )() == 0
@test toInt( fib(𝟙) )() == 1
@test toInt( fib(𝟚) )() == 1
@test toInt( fib(𝟛) )() == 2
@test toInt( fib(𝟜) )() == 3
@test toInt( fib(𝟝) )() == 5
@test toInt( fib(𝟞) )() == 8
@test toInt( fib(𝟟) )() == 13
@test toInt( fib(𝟠) )() == 21
@test toInt( fib(𝟡) )() == 34
```

"There. It is all as foretold." You have Tim ensnared in your spell now. Captivated. The job is yours!

"Ok. Whatever. I was going to ask how you'd optimize this, but&mdash;"

You've traversed this path before, you know where it leads. You preempt Tim:

```julia
julia> @code_native toInt(fib(𝟡))()
	.section	__TEXT,__text,regular,pure_instructions
	.globl	"_julia_#66_163"                
"_julia_#66_163":                       
	.cfi_startproc
	mov	w0, #34
	ret
	.cfi_endproc
```

"Yeah, we're done here." Tim makes a hasty retreat towards the door.

Perhaps you have been chosen. You harbor a good feeling about this one.
## Notes

The inspiration for this note came from the combination of [Aphyr's blog](https://aphyr.com/tags/interviews) series of technical interview parodies with this blog on [Lambda calculus in JavaScript](https://www.willtaylor.blog/combinators-and-church-encoding-in-javscript/)

[Combinators](https://en.wikipedia.org/wiki/SKI_combinator_calculus) are the slightly unintuitive alternative to the lambda calculus. Using nothing but two functions (in this case the `S`, and `K` combinators) you can program pretty much anything. To keep code size down when implementing tuples I also elected to use the `C`, `I` and `B` combinators. Please forgive this.

For some philosophical background to these (the ~I~diot, ~K~estrel, ~C~ardinal and ~B~luebird) see the fantastic book [To Mock a Mockingbird](https://en.wikipedia.org/wiki/To_Mock_a_Mockingbird). 

And for the curious, here is the entire code so you don't have to copy and paste each segment. Like most languages with some degree of functional support, Julia uses lazy evaluation which means that [thunking](https://en.wikipedia.org/wiki/Thunk) was necessary to make this whole illusion hold.

For the astute; no, the Julia code decompilation above isn't strictly right. But since we are deep in the realms of theoretical languages I thought it was a small liberty to take: a pure `S` / `K` combinator language would have been forced to evaluate and reduce the expressions at compile-time not run-time and thus `fib(𝟡)` would have been O(1).

```julia
# Identity/Idiot, Kestrel, Cardinal and Bluebird
const I = x -> x
const K = x -> y -> x
const C = f -> x -> y -> f(y)(x)
const B = f -> g -> x -> f(g(x))

# Some boolean truths:
const T = K
const F = K(I)

# Tuples:
const pair = B(C)(C(I))
const first = C(I)(T)
const second = C(I)(F)
const next = n -> pair(F)(n)
const prev = n -> second(n)

# Natural numbers..
const 𝟘 = I
const 𝟙 = next(𝟘)
const 𝟚 = next(𝟙)
const 𝟛 = next(𝟚)
const 𝟜 = next(𝟛)
const 𝟝 = next(𝟜)
const 𝟞 = next(𝟝)
const 𝟟 = next(𝟞)
const 𝟠 = next(𝟟)
const 𝟡 = next(𝟠)

const isZero = n -> first(n)
const ifThenElse = I

# Apply, a derrivative of the Y-combinator:
const apply = n -> f -> x -> () -> 
    ifThenElse(isZero(n))(()->x)(()->f(apply(prev(n))(f)(x)()))()

const add = m -> n -> apply(m)(next)(n)()

# Fibonacci from pairs:
const nextFibPair = p -> pair(second(p))(add(first(p))(second(p)))
const fib = n -> first(apply(n)(nextFibPair)(pair(𝟘)(𝟙))())

# Unit tests:
using Test

const toInt = n -> () ->
    ifThenElse(isZero(n))(() -> 0)(() -> 1 + (toInt(prev(n)))())()

@test toInt( fib(𝟘) )() == 0
@test toInt( fib(𝟙) )() == 1
@test toInt( fib(𝟚) )() == 1
@test toInt( fib(𝟛) )() == 2
@test toInt( fib(𝟜) )() == 3
@test toInt( fib(𝟝) )() == 5
@test toInt( fib(𝟞) )() == 8
@test toInt( fib(𝟟) )() == 13
@test toInt( fib(𝟠) )() == 21
@test toInt( fib(𝟡) )() == 34
```

Feedback, insults, corrections and improvements greatly appreciated!
