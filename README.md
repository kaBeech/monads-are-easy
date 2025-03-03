# Monad Notes

I made this when the ideas of monads and related concepts finally clicked in my head.

My goal is not to provide a complete introduction to monads, but to summarize the
important points in a way that is easy to understand for someone who already has some
experience working with or learning about monads.

For a more complete introduction, please see the [sources](#Sources) below. Alternatively,
ask your local functional programming nerd/type theorist and they likely have even better
resources to recommend!

## Images

![monads_monoids](https://github.com/user-attachments/assets/77ddca7f-1223-44a8-b726-7ef3dc8b24f4)

![monoids](https://github.com/user-attachments/assets/9ba8ec53-e9f7-4800-b96c-3d78dfa0b802)

![monads](https://github.com/user-attachments/assets/efe6535a-d225-4bc8-b714-8b000566d91f)

## Pseudo-Haskell

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
    typeConstructor :: a -> M a
    typeConstructor x = M x

    -- Monads must implement.
    -- AKA typeConverter or return.
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
    safeNeck = safeHead x >>= safeHead
    safeNeck' = bind (safeHead x) safeHead

*For more information on the Maybe monad, see pretty much any introductory source on monads below, like [Learn You A Haskell](https://learnyouahaskell.github.io/a-fistful-of-monads.html#getting-our-feet-wet-with-maybe) or [Wikipedia](https://en.wikipedia.org/wiki/Monad_(functional_programming)#Overview)

## Sources

Some of these sources are more reliable than others, but I've found them all helpful in gaining a more complete understanding

- https://github.com/haskell/mtl
- https://github.com/haskell/random
- https://github.com/ekmett/free
- https://hackage.haskell.org/package/ghc-internal-9.1201.0/docs/src/GHC.Internal.Data.Maybe.html#maybe
- https://learnyouahaskell.github.io/
- https://en.wikibooks.org/wiki/Write_Yourself_a_Scheme_in_48_Hours
- https://www.youtube.com/watch?v=ENo_B8CZNRQ
- https://www.youtube.com/watch?v=VgA4wCaxp-Q
- https://en.wikipedia.org/wiki/Monad_(functional_programming)
- https://en.wikipedia.org/wiki/Monad_(category_theory)
- https://en.wikipedia.org/wiki/Monoid
- https://wiki.haskell.org/index.php?title=Monad
- https://www.youtube.com/watch?v=t1e8gqXLbsU
