---
title: "SKIing on the Technical Interview"
tags: [Aphyr, Interview, Combinators, Julia]
date: 2023-08-01
---

# SKIing on the Technical Interview







By Mark Dixon
## SKIing on the Technical Interview

You follow Tim into the well-lit chamber. A little too bright for your liking, you feel an aprehensive panic starting to set in. Perhaps the rumours you've heard about this little clan are true - your instincts tell you to back away but Tim is quicker and pushes you towards a chair.

"Ok, so, you've probably done this a thousand times before, judging by your beard," he begins, "so lets just get right to it."

Not sure if to take him seriously or not, your focus drifts up to Tim's neckbeard. He won't go far in this world. But sadly, you need his money.

Pointing to the modern-looking teletype in front of you, Tim continues: "Right, so lets start with something easy to get us warmed up. How about a function that returns the factorial of a number?"

You nod with a feeling of things finally moving towards your natural territory. You were bottle-fed factorials as a bairn back home in the north.

"What syntax does your tribe use?" you ask, remembering cousin Aisha's words of advice when adapting to the mortals' ways of working.

"Um, you mean language? We're primarily a Go company but feel free to use whatever you want really. But do try to be practical - nothing fancy, I just want to see how you'd work day to day problems."

Your aprehension is now replaced by confusion. Tim clearly understands that functions are the source of everything. How could that be fancy? Oh&hellip; you get it now. How do mortals code?

You run your hands over the keys --

"We have Emacs on this machine?!"

&mdash; and conjure up a file from the nether.

```julia
using SpecialFunctions
ᚠ = factorial
ᚠ(10)
```

"Wait, .jl? What language is that? Julia?" Tim is looking confused.

"You asked for practical," you reply patiently, "so I selected an Earth syntax. Perhaps you would have rather I followed the ways of Water?"

"Riiight.. but, you know, I was kind of hoping you would show how to build up a factorial function yourself and not just use a built-in functon," Tim responds. "Ok, let's try again. How about you write a function to compute the nth Fibonacci number. This time just stick to basic built-in language features you know? Show how you would build up the function from scratch."

You've never understood the mortals, and the smell of this one is starting to get to you. 

"So no re-using any tomes of wisdom? Build it up from a minimum set of the elements?" you ask.

Tim nods: "Sure, yeah the elements, whatever. You can keep using Julia if you like."

Now you feel that you are moving on the same currents as Tim. Perhaps you underestimated him? No, but you know exactly how to do this kind of work. Just the function operator. You can do that in your sleep. Clearing the buffer, you close your eyes as you conjure up the oldest runes of them all:

```julia
const ᛝ = ᚷ->ᚷ
const ᚴ = ᚷ->ᛉ->ᚷ
const ᛣ = ᚠ->ᚷ->ᛉ->ᚠ(ᛉ)(ᚷ)
const ᛒ = ᚠ->ᚵ->ᚷ->ᚠ(ᚵ(ᚷ))
```

"Woah! Hold on there," interrupts Tim. "I've seen this sort of trick before. Stick to the Roman alphabet please."

Heathen! You briefly contemplate leaving the chamber, but then remember that you need this more than Tim needs you. Curse this age.

Your fingers flash over the keys as you reweave your spell into the heathen's tounge. A silent prayer on your lips &mdash; hopefuly the runes are powerful enough to still work their magic. You've never tried using them this way before:

```julia
const I = x -> x
const K = x -> y -> x
const C = f -> x -> y -> f(y)(x)
const B = f -> g -> x -> f(g(x))
```

"Ok, that's better," Tim breathes out. "For a second there I thought you were one of those crazies we get from time to time. But what does this have to do with the Fibonacci sequence?"

"Patience, oh Tim." You try to calm him with your Voice. "We will get to that shortly. But first we need some elemental help."

Your fingers speed up, you don't want Tim to dispell the Flow taking over you.

"First the contradictory forces of Fire and Ice. Muspelheim and Niflheim. Or the duality of right and wrong as you would say in your tounge."

```julia
const T = K
const F = K(I)
```

"Oh, true and false! Ok but try to name your variables a bit more descriptively," Tim is definitely looking a tad worried now. "And explain what you are doing as you go along, I need to understand how you think." 

You don't want to displease him, so you immediately adapt to these new demands. Not that Tim seems to understand the difference between variables and the fundaments of the cosmic void. This type of art you are performing is not called 'thinking' - merely letting Yggdrasil use you as a vessel.

"I don't really think, I&hellip; thunk in your speach maybe? So things are always strongest in twos," you try to explain your ways. "I believe you would call them pairs?"

```julia
const pair = B(C)(C(I))
const first = C(I)(T)
const second = C(I)(F)
const next = n -> pair(F)(n)
const prev = n -> second(n)
```

"Ah, ok, so you are building a tuple class? I thought we were going to start with something simpler."

You ignore Tim. The incantations are starting to take over now. Your Flow is strong. What next? Numbers! Of course, you need the naturals:

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

"Ok, that's a bit odd, but I think I see where this is heading," Tim tries to convince you he is still in control here. "But Julia doesn't have lazy evaluation right?" Fool. Do you think you can outsmart me?

One last burst and now you just need to thunk some final forms for this spell to be complete:

```julia
const isZero = n -> first(n)
const ifThenElse = I
const apply = n -> f -> x -> () -> 
    ifThenElse(isZero(n))(()->x)(()->f(apply(prev(n))(f)(x)()))()
const add = m -> n -> apply(m)(next)(n)()
```

There, you lean back satisfied. That should just about do it. You must really be pleasing Tim now &mdash; just functions all the way down to the roots.

Tim is leaning his head to the side squinting at the screen. A long silence ensues.

"Tim? So Fibonacci right? Even those sacred numbers obey the ancient laws of dualism," you try to reassure him. "So all we need to do now is to ask for the right pair&hellip;"

Tim nods. That's your signal. Time to cast the spell:

```julia
const nextFibPair = p -> pair(second(p))(add(first(p))(second(p)))
const fib = n -> first(apply(n)(nextFibPair)(pair(𝟘)(𝟙))())
```

Your Flow recedes. The tension in the room is starting to dissipate.

"Ok. Great. Um," he mumbles. "So how do we know if this works? Can you, like, write some unit tests or something?"

Stupid! Of course. Again, you have neglected Aisha's advice. These mortals never put their trust in the gods. They need their own proofs.

"Sure," you say, trying to appease. "But then I'd have to use some more language features.. like maybe integers and the plus operator, hope thats OK?"

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

"There. It is all true." You have Tim in your spell now. Trapped. The job is yours!

"Ok. Whatever. I was going to follow up by asking how you would optimise this, but&mdash;"

You've done this before, you know where it is leading. You preempt Tim:

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

"Yeah, we're done here." Tim makes a run for the door.

Perhaps you have been hired. You feel good about this one.
## Notes

The inspiration for this note came from a combination of [Aphyr's blog](https://aphyr.com/posts) series of technical interview parodies and [https://www.willtaylor.blog/combinators-and-church-encoding-in-javscript/](https://www.willtaylor.blog/combinators-and-church-encoding-in-javscript/)

[Combinators](https://en.wikipedia.org/wiki/SKI_combinator_calculus) are the slightly unintuitive alternative to the lambda calculus. Using nothing but two functions (in this case the S, and K combinators) you can build prett much anything. To keep code size down when implementing tuples I also elected to use the B combinator. For some philosophical background to these (the Idiot, Kestrel, Cardinal and Bluebird) see the fantastic book [To Mock a Mockingbird](https://en.wikipedia.org/wiki/To_Mock_a_Mockingbird). 

And for the curious, here is the entire code so you don't have to copy and paste each segment. Like many languages with some degree of functional support, Julia uses lazy evaluation which means that [thunking](https://en.wikipedia.org/wiki/Thunk) was necessary.

For the astute; no, the Julia code decompilation isn't really right. But since we are anyways in the realms of theoretical languages I thought it was a small liberty to take. A pure S/K combinator language would have been forced to evaluate and reduce the expressions at compile-time not run-time.

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
