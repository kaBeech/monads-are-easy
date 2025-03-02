# Monad Notes

![monads](https://github.com/user-attachments/assets/4b8c69bf-f019-465f-843e-535882794a2d)

    Functor: a -> b

    Endofunctor: a -> a

                        Effects!
                           |
                           v  
    Monad (a) = Monoid (a -> a)

    Monoid:
      Type: a
      Operator: op :: a -> a -> a
        where
          op(a1, op(a2, a3)) == op(op(a1, a2), a3)
      ID: id :: a
        where
          op(a1, id) == a1
