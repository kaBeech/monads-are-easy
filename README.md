# Monad Notes

## Images

![monads](https://github.com/user-attachments/assets/4916a6de-ccd0-4d90-be88-67ab30a455c7)

![monoids](https://github.com/user-attachments/assets/9ba8ec53-e9f7-4800-b96c-3d78dfa0b802)

![monads2](https://github.com/user-attachments/assets/540a3f0f-eac3-4a3f-b454-9a641e6e5c93)

## Pseudo-Haskell

    type Functor a b = a -> b

    type Endofunctor a = a -> a

                      Extra info/effects!
                               |
                               v  
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
    -- one element
    safeHead :: [a] -> Maybe a
    safeHead [] = Nothing
    safeHead (x : _) = Just x

    -- Example: Chaining binds.
    -- Both these examples give the second element of a 
    -- list with at least two elements
    safeNeck = safeHead x >>= safeHead
    safeNeck' = bind (safeHead x) safeHead
