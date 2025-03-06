# Monad Notes

![monads_monoids](https://github.com/user-attachments/assets/ec86a221-04d3-4b26-aa75-56eed2b872d7)

This is a page to collate my notes as I work to solidify in my mind what monads really are.
Once my understanding stablilizes I plan to convert this into more of a cheatsheet.

I hope it helps show how simple these concepts are!

If this seems like a long read, it's because it gives the same basic info in like 20
different ways! Feel free to jump around to whichever parts seem helpful and stop
reading once you feel you've got it =)

My goal is not to provide a complete introduction to monads, but to summarize the
important points in a way that is easy to understand for someone who already has some
experience working with or learning about monads.

For a more complete introduction, please see the [sources](#Sources) below. Alternatively,
ask your local functional programming nerd/type theorist and they likely have even better
resources to recommend!

## Summary

- Update: a monad is just an endofunctor that's also a monoid in the same category. Idk why
    Saunders couldn't have just said that haha. This page will likely get updated gradually.

A monadic type is the base type plus some structure which may contain extra information/effects.

Having a monad means that:

1. You have a type constructor that builds a monadic type `M (S a)` for elements of type `a` in
     structure `S` (which may have some extra information/effects)
  
2. You have a function that takes an `S a` and puts it into a monadic type `M (S a)`, AKA a type
     converter or return
   
3. You have a function that chains together operations performed upon an `M (S a)`, AKA a combinator
     or bind

4. You can use points 2 and 3 together to perform operations on elements of the monadic type that
     affect either only the elements or only the structure

## Images

"A monad (in `a`) is a monoid in the category of endofunctors (of `a`)." Since it's a monoid
(see below for more), we know we'll get something of the same (monadic) type out of the endofunctors
**and can safely hide some extra information/effects inside their structure/execution pipelines**:

![monads_monoids](https://github.com/user-attachments/assets/44a31819-e1c4-471f-8940-2f5e9c629517)

A monoid is a set combined with a binary associative operator and an identitiy element. Each monoid
has a type of elements within the set (this set type could be anything), an operator that exhibits
totality and associativity, and an element that exhibits identity when used with that operator:

![monoids](https://github.com/user-attachments/assets/d13d4c3a-18a3-440f-8126-1fb48700215b)

A monad is a monoid whose set type is a type of endofunctor. Each monad has a type constructor, a
bind operator, and a return operator:

![monads](https://github.com/user-attachments/assets/ecd2e356-944f-4035-be6a-e42743e4bd04)

## Basic Review

### Associativity

- ((f ∘ g) ∘ h)(x) = (f ∘ (g ∘ h))(x)

#### Examples of Associativity

    (x + y) + z = x + (y + z)

    (x * y) * z = x * (y * z)

#### Anti-Example of Associativity

    (x / y) / z != x / (y / z)

### Identity

- (f ∘ i)(x) = x
- (i ∘ f)(x) = x

#### Examples of Identity

    x + 0 = x

    1 * x = x

### Totality

- T -> T

#### Examples of Totality

    Integer + Integer = Integer

    Float * Float = Float

#### Anti-Example of Totality

    Integer / Integer = Float
    
## Compact Pseudo-Haskell

    class Functor S where  
        fmap :: (a -> b) -> S a -> S b  

    class Endofunctor S where  
        fmap :: (a -> a) -> S a -> S a  

    -- Monad a == Monoid (S a)

    class Monoid a where
        op :: a -> a -> a
        op x (op y z) == op (op x y) z
        id :: a
        op id x = x
        op x id = x

    class Monad a where
        TypeConstructor :: a -> M a
        TypeConstructor x = M x
        `>>=` :: M a -> a -> M b) -> M b
        M x >>= f = f x :: M b
        M x >>= (\x' -> (f x' >>= g)) == (M x >>= f) >>= g :: M c
        return :: a -> M a
        return x >>= f == f x :: M a
        M x >>= return == M x :: M a

## Verbose Pseudo-Haskell

    -- | A functor is something you can map over.
    --   A list is an example of a functor
    class Functor S where  
        fmap :: (a -> b) -> S a -> S b

    -- | An endofunctor is like a functor but elements in the output are the same
           type as elements in the input.
         A deck of cards (for the sake of example, ranked in order [2..K,A] and
           [C,D,H,S], no Jokers), is an example of an endofunctor. If you have a 
           function that takes an input card and returns the next card in ranked 
           order (or with 2C if the input card is AH), applying this function to a
           deck of cards would yield a deck of cards. If you have a function that
           takes an input card and returns a dollar bill, however, applying it to 
           a deck of cards would yield a stack of dollar bills, not a deck of cards.
    class Endofunctor S where  
        fmap :: (a -> a) -> S a -> S a 

    --   Structure - potentially 
    --      extra info/effects!
    --                    |
    --                    v  
    -- Monad a == Monoid (S a)
    
    class Monoid a where
        -- | Totality (per type definition)
        op :: a -> a -> a
        -- | Associativity
        op x (op y z) == op (op x y) z
        id :: a
        -- | Left identity
        op id x = x
        -- | Right identity
        op x id = x

    class Monad a where
        TypeConstructor :: a -> M a
        TypeConstructor x = M x
        -- | AKA combinator, map, or flatmap.
        --   Congruent to op in Monoid.
        --   Totality (per type definition)   
        bind :: M a -> (a -> M b) -> M b
        bind (M x) f = f(x) :: M b
        -- | Associativity
        bind (M x) (\x' -> (bind (f(x')) g)) == bind (bind (M x) f) g
        -- | Alternative bind - infix of above
        --   Totality (per type definition)   
        `>>=` :: M a -> (a -> M b) -> M b
        M x >>= f = f x :: M b
        -- | Associativity
        M x >>= (\x' -> (f x' >>= g)) == (M x >>= f) >>= g :: M c
        -- | AKA type converter or unit.
        --   Congruent to id in Monoid
        return :: a -> M a
        -- | Left identity
        return x >>= f == f x :: M a
        -- | Right identity
        M x >>= return == M x :: M a
            
    -- | Example: Function returning a monadic type.
    --   Inflates an imaginary (i.e. actually inflating the balloon as a side
    --   effect is not modeled here) balloon by 5 pounds per square inch.
    --   Returns Nothing if the balloon pops.
    --   See note* below if this doesn't make sense
    inflateBalloon :: Int -> Maybe Int
    inflateBalloon psi
      | lb <= 50 = Just (5 + psi)
      | otherwise = Nothing

    -- | Example: Chaining binds.
    --   Inflates the imginary balloon twice, for a total of 10 additional
    --   pounds per square inch.
    --   Returns Nothing if the balloon pops during either inflation step
    dangerouslyInflateBalloon :: Int -> Maybe Int
    dangerouslyInflateBalloon xs = inflateBalloon x >>= inflateBalloon
    dangerouslyInflateBalloon' :: Int -> Maybe Int
    dangerouslyInflateBalloon' xs = bind (inflateBalloon x) inflateBalloon

    -- | Example: Adding Effects.
    --   Deflates the imaginary balloon by 5 pounds per square inch and
    --   prints the new psi to stdout
    deflateBalloonWithMonitor :: Int -> IO Int
    deflateBalloonWithMonitor psi 
        | psi <= 0 = do
            let psi' = 0
            print psi'
            return psi'
        | otherwise = do
            let psi' = psi - 5
            print psi'
            return psi'

*For more information on the Maybe monad, see pretty much any introductory
source on monads below, like [Learn You A Haskell](https://learnyouahaskell.github.io/a-fistful-of-monads.html#getting-our-feet-wet-with-maybe)
or [Wikipedia](https://en.wikipedia.org/wiki/Monad_(functional_programming)#Overview)

## Sources

Some of these sources are more reliable than others, but I've found them
all helpful in gaining a more complete understanding. Notably, the two videos
at the top contain some inaccuracies, but they still can be a good starting point for
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
- [Functor](https://hackage.haskell.org/package/ghc-internal-9.1201.0/docs/src/GHC.Internal.Data.Functor.html)
- [Monoid](https://hackage.haskell.org/package/ghc-internal-9.1201.0/docs/src/GHC.Internal.Data.Monoid.html)
- [Monad](https://hackage.haskell.org/package/ghc-internal-9.1201.0/docs/src/GHC.Internal.Control.Monad.html)
- [Maybe](https://hackage.haskell.org/package/ghc-internal-9.1201.0/docs/src/GHC.Internal.Data.Maybe.html#maybe)
- [mtl](https://github.com/haskell/mtl)
- [random](https://github.com/haskell/random)
- [free](https://github.com/ekmett/free)
