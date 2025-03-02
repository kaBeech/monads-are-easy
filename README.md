# Monad Notes

![monads](https://github.com/user-attachments/assets/4b8c69bf-f019-465f-843e-535882794a2d)

    type Functor a b = a -> b

    type Endofunctor a = a -> a

                        Effects!
                           |
                           v  
    type Monad (a) = Monoid (a -> a)

    type Monoid a
        where
          op :: a -> a -> a
            where
              op(a1, op(a2, a3)) == op(op(a1, a2), a3)
          id :: a
            where
              op(a1, id) == a1

    -- Monads must implement
    typeConstructor :: a -> M a
    typeConstructor x = M x

    -- Monads must implement
    -- AKA typeConverter
    unit :: a -> M a
        where
            -- Left identity
            unit(x) >>= f == f(x)
            -- Right identity
            M x >>= unit == M x

    -- Monads must implement
    -- AKA combinator
    -- Type definition implies totality
    bind :: M a -> (a -> M b) -> M b
    bind (M x) f = f(x) :: M b
        where
            -- Associativity
            bind (M x) (\x -> (bind (f(x)) g)) == bind (bind (M x) f) g

    -- Alternative bind - infix of above
    `>>=` :: M a -> (a -> M b) -> M b
    M x >>= f = f(x) :: M b
        where
            -- Associativity
            M x >>= (\x -> (f(x) >>= g)) == (M x >>= f) >>= g
            
    -- Example of a function using monadic patterns
    -- Gives the first element of a list with at least
    -- one element
    safeHead :: [a] -> Maybe a
    safeHead [] = Nothing
    safeHead (x : _) = Just x
    
    -- Both these examples give the second element of a 
    -- list with at least two elements
    safeNeck = safeHead x >>= safeHead
    safeNeck' = bind (safeHead x) safeHead
