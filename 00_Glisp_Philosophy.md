# GLISP – TL;DR Philosophy - (Deep topic)

> **Authority:** This document is the authoritative technical description. No abstractions or proposals beyond what is written here.

--- 

**Reason, Audacity and Madness**:
```
"And those who were seen dancing were considered mad by those who couldn't hear the music.",
Perhaps by those who can hear the music but can't understand the lyrics; who knows people's hearts?

"Reason, audacity, and madness are relative concepts that remain fixed only in the minds of the observer,
we have enough people to judge things, but if you're an engineer, have a stronger desire to build things." -- Wilton Lazary.
```

**Engineer**:
```
Anyone who builds, maintains, or fixes things and whose mind is wired toward practical problem-solving, a cognitive property.
```

**Programming language as a tool for engineers**:
```
Remember that a programming language is a model, and all models are flawed at some level and cannot fully
represent the richness of the real world.
Glisp descriptions are tools to create a flexible programming language.
It can be used by any being or entity with any worldview or preference (including from other worlds, e.g., aliens).
The only requisite is that it be written in common-ground language (US English - by now).
It does not incorporate, deny, or enforce any view or preference—including moral, cultural, societal, governance, or public-matter stances,
nor involve itself in conflict creation or pacification; it is only a tool for engineers.
If some being or entity does not feel comfortable with or capable of using this tool, use another tool or none.
This tool aims to be as style-neutral as possible; so it uses the record in directives to represent otherwise composite names.
Technical descriptions should be precise and unambiguous, no fluff, no sugarcoating,
just explanatory facts and rules, preventing different readers from making different assumptions about the same information.
The intention is to create a language that can be used by engineers to type commands and build software; other uses are unintentional.
```

**No worship around the tool**:
```
The same tool that, in the hands of a skilled professional, can work wonders may,
in the hands of another equally or more skilled professional, not work as well because it causes calluses, consequently,
this second professional may prefer more ergonomic tools for their body/brain type.
Likewise, programming languages can work wonders for some and cause calluses on the body/brain for others.
```

**The fundamental laws**:
```
Engineers, if they dig deep enough, will eventually come across the fundamental laws of physics, especially the laws of thermodynamics,
which, interestingly, include the zeroth law. The practical world (that is, where the engineers operate) obeys the laws of thermodynamics.

With the thermal/energy formalism aside and assuming the symmetry, reflexivity and transitivity as core ideas:
  Equilibrium/Equality: if A matches C and B matches A, then B matches C (zeroth).

  Configurations/Transformations: everything is the same stuff in new forms; we rearrange (first).

  Direction/Inescapability: With some regularity, things tend to follow a natural direction and/or spread out,
  there are no free cycles, and there may be costs to redirecting, reorganizing, and concentrating things (second).

  Absolute-zero/All-settled: when all things will be calm and resolved; we are not there yet (third).
```

**The fundamental laws and GLISP Design**:
```
GLISP's design is based on thermodynamics:
Just as thermodynamics provides fundamental laws that govern energy transformations
without dictating specific configurations, GLISP provides core parsing rules that enable
various surface syntaxes without prescribing one style.

The fundamental rules in GLISP:
1. Mechanical transformation - Parser transforms characters arrangements to AST deterministically.
2. Near interactions (short-range forces) - No long range lookahead or backtracking required.
3. Preserved structure - Source information flows through transformations.
4. Composable transformations - Rules combine predictably.

From these, you can build multiple styles:
  # Shell style
  ls -la | grep error

  # Functional style
  data |> map(f) |> filter(pred)

  # OOP style
  obj.method().field

  # Lisp style
  (map f (filter pred data))

All emerge from the same core rules, just as different states of matter emerge from the same thermodynamic laws.
The parser doesn't care about "style" - it just applies transformations mechanically.

The key is that transformations are defined, not emergent. When 'f x' becomes '(f x)',
it's because a specific rule fired, not because the parser "figured out" that f is probably a function.
This maintains predictability while allowing flexibility.

This is why GLISP can unify shell scripting and general programming - not through special cases,
but through fundamental rules that naturally accommodate both.
```

**Guardrails and the foot**:
```
Walls meant to protect often block, guardrails protect without denying access to the core.
The engineer must be free—and responsible—to touch the core; with reason, audacity or madness.
"Accept the cons when the foot presses on, beyond the guardrails, you are on your own." -- Wilton Lazary.
```

**The Wall**:
```
Let's be "concrete": walls exist for a variety of reasons:
  Perhaps a company needs to standardize and control its environment.
  Perhaps some people need to be protected from others or themselves.
  Perhaps a culture needs to preserve its identity.
  Perhaps a powerful tool needs to be controlled to prevent misuse.
  Perhaps all of this together, and maybe even more, who knows?

In any case, the tool is not here to dictate what is right or wrong.
Other tools can be built on top of it to achieve these goals,
such as fixing the style based on the file extension.
```

**No "Design" Morality**:
```
GLISP explicitly rejects the traditional "hygiene" debate. There is no moral superiority in design—there are no "clean" or "dirty"
approaches, nor distinctions between "hygienic" and "unhygienic." These terms introduce unnecessary moral judgments into engineering
decisions. GLISP favors styles ranging from command line to OOP, passing by DOP and FFP, not because they are a better sequence,
but simply because they are a natural and fluid order, from simplicity to complexity. The engineer should be free to begin and use any of
them freely, without judgments beyond those immediately related, such as design rules and organizational style,
or personal paranoia imposed by the engineer on himself ...
```
