# Monads Are Easy!

I made this when the ideas of monads and related concepts "fully" clicked in my head.

I hope this helps show how simple these concepts really are!

My goal is not to provide a complete introduction to monads, but to summarize the
important points in a way that is easy to understand for someone who already has some
experience working with or learning about monads.

For a more complete introduction, please see the [sources](#Sources) below. Alternatively,
ask your local functional programming nerd/type theorist and they likely have even better
resources to recommend!

## Summary

A monadic type is the base type plus some extra information/effects.

Having a monadic type `M a` simply means that you can make some functions that take an `a`
and return an `a`, potentially with some extra information/effects that can be safely ignored
while still guaranteeing some essential properties (i.e. associativity, identity, and
totality*).

*Totality is redundant here, as it's already implied by taking and returning an `a`.

## Images

"A monad (in `a`) is a monoid in the category of endofunctors (of `a`)":

![monads_monoids](https://github.com/user-attachments/assets/4d4e2715-8fca-4eb9-9536-9e9fe0f0e956)

Monoids:

![monoids](https://github.com/user-attachments/assets/d13d4c3a-18a3-440f-8126-1fb48700215b)

Monads:

![monads](https://github.com/user-attachments/assets/4a0465fc-cad7-44e4-a2cd-989ef464cb7a)

## Compact Pseudo-Haskell

    type Functor a b = a -> b

    type Endofunctor a = a -> a

    type Monad (a) = Monoid (a -> a)

    type Monoid a
        where
          op :: a -> a -> a
            where
              op(x, op(y, z)) == op(op(x, y), z)
          id :: a
            where
              op(id, x) == x
              op(x, id) == x

    TypeConstructor :: a -> M a
    TypeConstructor x = M x

    unit :: a -> M a
        where
            unit(x) >>= f == f(x)
            M x >>= unit == M x
            
    `>>=` :: M a -> (a -> M b) -> M b
    M x >>= f = f(x) :: M b
        where
            M x >>= (\x' -> (f(x') >>= g)) == (M x >>= f) >>= g

## Verbose Pseudo-Haskell

    type Functor a b = a -> b

    type Endofunctor a = a -> a

    -- (Potentially) extra info/effects!
    --                          |
    --                          v  
    type Monad (a) = Monoid (a -> a)

    type Monoid a
        where
          -- Totality (per type definition)
          op :: a -> a -> a
            where
              -- Associativity
              op(x, op(y, z)) == op(op(x, y), z)
          id :: a
            where
              -- Left identity
              op(id, x) == x
              -- Left identity
              op(x, id) == x

    -- Monads must implement
    TypeConstructor :: a -> M a
    TypeConstructor x = M x

    -- Monads must implement.
    -- AKA type converter or return.
    -- Congruent to id in Monoid
    unit :: a -> M a
        where
            -- Left identity
            unit(x) >>= f == f(x)
            -- Right identity
            M x >>= unit == M x

    -- Monads must implement.
    -- AKA combinator, map, or flatmap.
    -- Congruent to op in Monoid.
    -- Totality (per type definition)   
    bind :: M a -> (a -> M b) -> M b
    bind (M x) f = f(x) :: M b
        where
            -- Associativity
            bind (M x) (\x' -> (bind (f(x')) g)) == bind (bind (M x) f) g

    -- Alternative bind - infix of above
    -- Totality (per type definition)   
    `>>=` :: M a -> (a -> M b) -> M b
    M x >>= f = f(x) :: M b
        where
            -- Associativity
            M x >>= (\x' -> (f(x') >>= g)) == (M x >>= f) >>= g
            
    -- Example: Function returning a monadic type.
    -- Gives the first element of a list with at least
    -- one element.
    -- See note* below if this doesn't make sense
    safeHead :: [a] -> Maybe a
    safeHead [] = Nothing
    safeHead (x : _) = Just x

    -- Example: Chaining binds.
    -- Both these examples give the second element of a 
    -- list with at least two elements
    safeNeck x = safeHead x >>= safeHead
    safeNeck' x = bind (safeHead x) safeHead

*For more information on the Maybe monad, see pretty much any introductory
source on monads below, like [Learn You A Haskell](https://learnyouahaskell.github.io/a-fistful-of-monads.html#getting-our-feet-wet-with-maybe)
or [Wikipedia](https://en.wikipedia.org/wiki/Monad_(functional_programming)#Overview)

## Sources

Some of these sources are more reliable than others, but I've found them
all helpful in gaining a more complete understanding. Notably, the two videos
at the top contain some inaccuracies, but they can be a good starting point for
wrapping your brain around the concepts in a practical way.

- [A monad is a monoid in the category of endofunctors. Whats the problem? #SoMe2](https://www.youtube.com/watch?v=ENo_B8CZNRQ)
- [What is a monad? (Design Pattern)](https://www.youtube.com/watch?v=VgA4wCaxp-Q)
- [Wikipedia: Monad (functional programming)](https://en.wikipedia.org/wiki/Monad_(functional_programming))
- [Wikipedia: Monad (category theory)](https://en.wikipedia.org/wiki/Monad_(category_theory))
- [Wikipedia: Monoid](https://en.wikipedia.org/wiki/Monoid)
- [Learn You A Haskell (Community Edition)](https://learnyouahaskell.github.io/)
- [Write Yourself a Scheme in 48 Hours (Wikibooks Edition)](https://en.wikibooks.org/wiki/Write_Yourself_a_Scheme_in_48_Hours)
- [Haskell Wiki: Monad](https://wiki.haskell.org/index.php?title=Monad)
- [What is a Monad? - Computerphile](https://www.youtube.com/watch?v=t1e8gqXLbsU)
- [Maybe](https://hackage.haskell.org/package/ghc-internal-9.1201.0/docs/src/GHC.Internal.Data.Maybe.html#maybe)
- [mtl](https://github.com/haskell/mtl)
- [random](https://github.com/haskell/random)
- [free](https://github.com/ekmett/free)
