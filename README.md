# GENERIC-CL

GENERIC-CL provides a generic function wrapper over various functions in the Common Lisp standard, such as equality predicates and sequence operations. The goal of the wrapper is to provide a standard interface to common operations, such as testing for the equality of two objects, which is extensible to user-defined types.


# Table of Contents

1.  [GENERIC-CL](#org5740a7b)
    1.  [Usage](#org9e34803)
    2.  [Generic Interfaces](#orgdbf0f77)
        1.  [Equality](#org264eb97)
        2.  [Comparison](#org2850589)
        3.  [Arithmetic](#orge64b7e4)
        4.  [Objects](#orga8fd3ec)
        5.  [Iterator](#orga651db4)
        6.  [Collector](#collector)
        7.  [Sequences](#orgf85810d)
        8.  [Generic Hash-Tables](#orga1dad3f)
        9.  [Set Operations](#org5c19cb7)
        10. [Math Functions](#org56f8601)
    3.  [Optimization](#org2af4923)


<a id="org9e34803"></a>

## Usage

The generic function interface is contained in the `GENERIC-CL` package. This package should be used rather than `COMMON-LISP`, as it shadows the symbols, in the `COMMON-LISP` package, which name a function for which there is a generic function wrapper. The `GENERIC-CL` additionally reexports the remaining non-shadowed symbols in `COMMON-LISP`.


<a id="orgdbf0f77"></a>

## Generic Interfaces

The generic function interface consists of the following functions divided into the following categories:


<a id="org264eb97"></a>

### Equality

The equality interface provides functions for testing objects for equality.

1.  EQUALP

    Generic Function: `EQUALP A B`
    
    Returns true if object `A` is equal to object `B`.
    
    Has methods specialized on the following types:
    
    -   `NUMBER NUMBER`
        
        Returns true if `A` and `B` represent the same numeric value, as per `CL:=`.
    
    -   `CHARACTER CHARACTER`
        
        Returns true if `A` and `B` represent the same character, as per `CL:CHAR=`.
    
    -   `CONS CONS`
        
        Returns true if the `CAR` of `A` is equal (by `EQUALP`) to the `CAR` of `B` and if the `CDR` of `A` is equal (by `EQUALP`) to the `CDR` of `B`.
    
    -   `VECTOR VECTOR`
        
        Returns true if `A` and `B` are vectors of the same length and each element of `A` is equal (by `EQUALP`) to the corresponding element of `B`.
    
    -   `ARRAY ARRAY`
        
        Multi-dimensional arrays.
        
        Returns true if `A` and `B` have the same dimensions and each element of `A` is equal (by `EQUALP`) to the corresponding element of `B`.
    
    -   `STRING STRING`
        
        Returns true if both strings are equal (by `CL:STRING=`).
    
    -   `PATHNAME PATHNAME`
        
        Returns true if both `PATHNAME` objects are functionally equivalent, as per the `PATHNAME-EQUAL` function from the `CL-FAD` library.
    
    -   `T T`
        
        Default method.
        
        Returns true if `A` and `B` are the same object, as per `CL:EQ`.

2.  =

    Function: `= X &REST XS`
    
    Returns true if all objects in `XS` are equal (by `EQUALP`) to `X`.

3.  /=

    Function: `= X &REST XS`
    
    Returns true if at least one object in `XS` is not equal (by `EQUALP`) to `X`.


<a id="org2850589"></a>

### Comparison

The comparison interface provides functions for comparing objects in terms of greater than, less than, greater than or equal to and less than or equal to relations.

1.  LESSP

    Generic Function: `LESSP A B`
    
    Returns true if object `A` is less than object `B`.
    
    It is sufficient to just implement this function, for user-defined types, as the rest of the comparison functions have default (`T T`) methods which are implemented in terms of `LESSP`.
    
    Has the following methods:
    
    -   `NUMBER NUMBER`
        
        Returns true if the numeric value of `A` is less than the numeric value of `B`, by `CL:<`.
    
    -   `CHARACTER CHARACTER`
        
        Returns true if the character code of `A` is less than the character code of `B`, by `CL:CHAR<`.
    
    -   `STRING STRING`
        
        Returns true if the string `A` is lexicographically less than `B`, by `CL:STRING<`.

2.  LESS-EQUAL-P

    Generic Function: `LESS-EQUAL-P A B`
    
    Returns true if object `A` is less than or equal to object `B`.
    
    Has the following methods:
    
    -   `NUMBER NUMBER`
        
        Returns true if the numeric value of `A` is less than or equal to the numeric value of `B`, by `CL:<=`.
    
    -   `CHARACTER CHARACTER`
        
        Returns true if the character code of `A` is less than or equal to the character code of `B`, by `CL:CHAR<=`.
    
    -   `STRING STRING`
        
        Returns true if the string `A` is lexicographically less than or equal to `B`, by `CL:STRING<=`.
    
    -   `T T`
        
        Returns true if either `A` is less than `B` (by `LESSP`) or `A` is equal to `B` (by ~EQUALP).
        
        ```lisp
        (or (lessp a b) (equalp a b))
        ```

3.  GREATERP

    Generic Function: `GREATERP A B`
    
    Returns true if object `A` is greater than object `B`.
    
    Has the following methods:
    
    -   `NUMBER NUMBER`
        
        Returns true if the numeric value of `A` is greater than the numeric value of `B`, by `CL:>`.
    
    -   `CHARACTER CHARACTER`
        
        Returns true if the character code of `A` is greater than the character code of `B`, by `CL:CHAR>`.
    
    -   `STRING STRING`
        
        Returns true if the string `A` is lexicographically greater than `B`, by `CL:STRING>`.
    
    -   `T T`
        
        Returns true if `A` is not less than or equal to `B`, by `LESS-EQUAL-P`.
        
        ```lisp
        (not (less-equal-p a b))
        ```

4.  GREATER-EQUALP

    Generic Function: `GREATER-EQUAL-P A B`
    
    Returns true if object `A` is greater than or equal to object `B`.
    
    Has the following methods:
    
    -   `NUMBER NUMBER`
        
        Returns true if the numeric value of `A` is greater than or equal to the numeric value of `B`, by `CL:>=`.
    
    -   `CHARACTER CHARACTER`
        
        Returns true if the character code of `A` is greater than or equal to the character code of `B`, by `CL:CHAR>=`.
    
    -   `STRING STRING`
        
        Returns true if the string `A` is lexicographically greater than or equal to `B`, by `CL:STRING>=`.
    
    -   `T T`
        
        Returns true if `A` is not less than `B`, by `LESSP`.
        
        ```lisp
        (not (lessp a b))
        ```

5.  COMPARE

    Generic Function: `COMPARE A B`
    
    Returns:
    
    -   **`:LESS`:** if `A` is less than `B`.
    -   **`:EQUAL`:** if `A` is equal to `B`.
    -   **`:GREATER`:** if `A` is greater than `B`.
    
    The default `T T` method returns:
    
    -   **`:LESS`:** if `(LESSP A B)` is true.
    -   **`:EQUAL`:** if `(EQUALP A B)` is true.
    -   **`:GREATER`:** otherwise.

6.  >

    Function: `< X &REST XS`
    
    Returns true if each argument is less than (by `LESSP`) than the following argument.

7.  <=

    Function: `<= X &REST XS`
    
    Returns true if each argument is less than or equal to (by `LESS-EQUAL-P`) than the following argument.

8.  <

    Function: `> X &REST XS`
    
    Returns true if each argument is greater than (by `GREATERP`) than the following argument.

9.  >=

    Function: `>= X &REST XS`
    
    Returns true if each argument is greater than or equal to (by `GREATER-EQUAL-P`) than the following argument.

10. MIN

    Function: `MIN X &REST XS`
    
    Returns the argument which is less than or equal to all other arguments, the actual comparisons are performed using `LESSP`. Any one of the arguments which satisfies this condition may be returned.

11. MAX

    Function: `MAX X &REST XS`
    
    Returns the argument which is greater than or equal to all other arguments, the actual comparisons are performed using `GREATERP`. Any one of the arguments which satisfies this condition may be returned.


<a id="orge64b7e4"></a>

### Arithmetic

The arithmetic interface provides generic functions for arithmetic operations.

1.  ADD

    Generic Function: `ADD A B`
    
    Returns the sum of `A` and `B`.
    
    Methods:
    
    -   `NUMBER NUMBER`
        
        Returns `(CL:+ A B)`.

2.  SUBTRACT

    Generic Function: `SUBTRACT A B`
    
    Returns the sum of `A` and `B`.
    
    Methods:
    
    -   `NUMBER NUMBER`
        
        Returns `(CL:- A B)`.

3.  MULTIPLY

    Generic Function: `MULTIPLY A B`
    
    Returns the sum of `A` and `B`.
    
    Methods:
    
    -   `NUMBER NUMBER`
        
        Returns `(CL:* A B)`.

4.  DIVIDE

    Generic Function: `DIVIDE A B`
    
    Returns the sum of `A` and `B`. If `A` is the constant `1`, the result should be the reciprocal of `B`.
    
    Methods:
    
    -   `NUMBER NUMBER`
        
        Returns `(CL:/ A B)`.

5.  NEGATE

    Generic Function: `NEGATE A`
    
    Returns the negation of `A`.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:- A)`.

6.  +

    Function: `+ X &REST XS`
    
    Returns the sum of all the arguments, computed by reducing over the argument list with the `ADD` function.
    
    If no arguments are provided, `0` is returned. If a single argument is provided it is returned.

7.  -

    Function: `- X &REST XS`
    
    Returns the difference of all the arguments, computed by reducing over the argument list with the `SUBTRACT` function.
    
    If only a single argument is provided the negation of that argument is returned, by the `NEGATE` function.

8.  \*

    Function: `* X &REST XS`
    
    Returns the product of all the arguments, computed by reducing over the argument list with the `MULTIPLY` function.
    
    If no arguments are provided, `1` is returned. If a single argument is provided it is returned.

9.  /

    Function: `/ X &REST XS`
    
    Returns the quotient of all the arguments, computed by reducing over the argument list with the `DIVIDE` function.
    
    If only a single argument is provided it, the reciprocal of that argument, `(DIVIDE 1 X)`, is returned.

10. 1+

    Generic Function: `1+ A`
    
    Returns `A + 1`.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:1+ A)`.
    
    -   `T`
        
        Returns `(ADD A 1)`.

11. 1-

    Generic Function: `1- A`
    
    Returns `A - 1`.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:1- A)`.
    
    -   `T`
        
        Returns `(SUBTRACT A 1)`.

12. INCF

    Macro: `INCF PLACE &OPTIONAL (DELTA 1)`
    
    Increments the value of `PLACE` by `DELTA`, which defaults to `1`, using the `ADD` function.
    
    Effectively:
    
    ```lisp
    (setf place (add place delta))
    ```

13. DECF

    Macro: `DECF PLACE &OPTIONAL (DELTA 1)`
    
    Decrements the value of `PLACE` by `DELTA`, which defaults to `1`, using the `SUBTRACT` function.
    
    Effectively:
    
    ```lisp
    (setf place (subtract place delta))
    ```

14. MINUSP

    Generic Function: `MINUSP A`
    
    Returns true if `A` is less than zero.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:MINUSP A)`.
    
    -   `T`
        
        Returns true if `A` compares less than `0`, by `LESSP`.
        
        ```lisp
        (lessp a 0)
        ```

15. PLUSP

    Generic Function: `PLUSP A`
    
    Returns true if `A` is greater than zero.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:PLUSP A)`.
    
    -   `T`
        
        Returns true if `A` compares greater than `0`, by `GREATERP`.
        
        ```lisp
        (greaterp a 0)
        ```

16. ZEROP

    Generic Function: `ZEROP A`
    
    Returns true if `A` is equal to zero.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:ZEROP A)`.
    
    -   `T`
        
        Returns true if `A` is equal to `0`, by `EQUALP`.
        
        ```lisp
        (equalp a 0)
        ```

17. SIGNUM

    Generic Function: `SIGNUM A`
    
    Returns `-1`, `0` or `1` depending on whether `A` is negative, `A` is equal to zero or `A` is positive.
    
    Methods:
    
    -   `SIGNUM`
        
        Returns `(CL:SIGNUM A)`.
    
    -   `T`
        
        Returns `-1` if `(MINUSP A)` is true, `0` if `(ZEROP A)` is true, `1` otherwise.

18. ABS

    Generic Function: `ABS A`
    
    Returns the absolute value of `A`.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:ABS A)`.
    
    -   `T`
        
        If `(MINUSP A)` is true, returns `(NEGATE A)` otherwise returns `A`.
        
        ```lisp
        (if (minusp a)
            (negate a)
            a)
        ```

19. EVENP

    Generic Function: `EVENP A`
    
    Returns true if `A` is even.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:EVENP A)`
    
    -   `T`
        
        Returns `(ZEROP (MOD A 2))`

20. ODDP

    Generic Function: `ODDP A`
    
    Returns true if `A` is odd.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:ODDP A)`
    
    -   `T`
        
        Returns `(NOT (EVENP A))`

21. FLOOR

    Generic Function: `FLOOR N D`
    
    Returns `N`, or `N/D` if `D` is provided, rounded towards negative infinity, as the first value, and the remainder of the division if any, as the second return value.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:FLOOR N D)` if `D` is provided otherwise returns `(CL:FLOOR N)`.

22. CEILING

    Generic Function: `CEILING N D`
    
    Returns `N`, or `N/D` if `D` is provided, rounded towards positive infinity, as the first value, and the remainder of the division if any, as the second return value.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:CEILING N D)` if `D` is provided otherwise returns `(CL:CEILING N)`.

23. TRUNCATE

    Generic Function: `TRUNCATE N D`
    
    Returns `N`, or `N/D` if `D` is provided, rounded towards zero, as the first value, and the remainder of the division if any, as the second return value.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:TRUNCATE N D)` if `D` is provided otherwise returns `(CL:TRUNCATE N)`.

24. ROUND

    Generic Function: `ROUND N D`
    
    Returns `N`, or `N/D` if `D` is provided, rounded towards the nearest integer. If `N`, or `N/D`, lies exactly halfway between two integers, it is rounded to the nearest even integer. The remainder of the division if any, is returned as the second return value.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:ROUND N D)` if `D` is provided otherwise returns `(CL:ROUND N)`.

25. MOD

    Generic Function: `MOD N D`
    
    Returns the remainder of the `FLOOR` operation on `N` and `D`.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:MOD N D)`.
    
    -   `T`
        
        Returns the second return value of `(FLOOR N D)`.

26. REM

    Generic Function: `REM N D`
    
    Returns the remainder of the `TRUNCATE` operation on `N` and `D`.
    
    Methods:
    
    -   `NUMBER`
        
        Returns `(CL:REM N D)`.
    
    -   `T`
        
        Returns the second return value of `(TRUNCATE N D)`.


<a id="orga8fd3ec"></a>

### Objects

The object interface provides miscellaneous functions for manipulating objects.

1.  COPY

    Generic Function: `COPY OBJECT &KEY &ALLOW-OTHER-KEYS`
    
    Returns a copy of `OBJECT`. If `OBJECT` is mutable, by some other functions, then the returned object should be distinct (not `EQ`) from `OBJECT`, otherwise the return value may be identical (`EQ`) to `OBJECT`.
    
    This function may accept additional keyword arguments which specify certain options as to how the object should be copied. Methods specialized on sequences accept a `:DEEP` keyword parameter, which if provided and is true a deep copy is returned otherwise a shallow copy is returned. If this applicable to a user-defined type, the `COPY` method for that type should also accept the `DEEP` keyword parameter.
    
    Methods:
    
    -   `CONS`
        
        Returns a new list which contains all the elements in `OBJECT`. If `:DEEP` is provided and is true, the list returned contains a copy of elements, copied using `(COPY ELEM :DEEP T)`.
    
    -   `VECTOR`
        
        Returns a new vector which contains all the elements in `OBJECT`. If `:DEEP` is provided and is true, the vector returned contains a copy of elements, copied using `(COPY ELEM :DEEP T)`.
    
    -   `ARRAY`
        
        Multi-Dimensional Arrays.
        
        Returns a new array, of the same dimensions as `OBJECT`, which contains all the elements in `OBJECT`. If `:DEEP` is provided and is true, the array returned contains a copy of elements, copied using `(COPY ELEM :DEEP T)`.
    
    -   `T`
        
        Simply returns `OBJECT`.
        
        This method is provided to allow sequences containing arbitrary objects to be copied safely, without signaling a condition, and to avoid having to write simple pass-through methods for each user-defined type.
        
        However this means that if the object, for which there is no specialized copy method, can be mutated, the constraints of the `COPY` function are violated.

2.  DEFSTRUCT

    Macro: `DEFSTRUCT OPTIONS &REST SLOTS`
    
    This is the same as `CL:DEFSTRUCT` however a `COPY` method for the structure type is automatically generated, which simply calls the structure's copier function. If the `(:COPIER NIL)` option is provided, the `COPY` method is not generated.

3.  COERCE

    Generic Function: `COERCE OBJECT TYPE`
    
    Coerces `OBJECT` to the type `TYPE`.
    
    The default (`T T`) method simply calls `CL:COERCE`.


<a id="orga651db4"></a>

### Iterator

The iterator interface is a generic interface for iterating over the elements of sequences and containers.

The iterator interface is implemented for lists, vectors, multi-dimensional arrays and `HASH-MAPS`.

1.  Base Iterator Type

    Structure: `ITERATOR`
    
    This structure serves as the base iterator type and is used as by certain methods of generic functions to specialize on iterators.
    
    All iterators should inherit from (include) `ITERATOR`, in order for methods which specialize on iterators to be invoked.
    
    **Note:** A `COPY` method should be implemented for user-defined iterators.

2.  Iterator Creation

    [ITERATOR](#iterator-func) is the high-level function for creating iterators, whereas [MAKE-ITERATOR](#make-iterator) AND [MAKE-REVERSE-ITERATOR](#make-reverse-iterator) are the generic functions to implement for creating iterators for user-defined sequence types.
    
    1.  MAKE-ITERATOR
    
        Generic Function: `MAKE-ITERATOR SEQUENCE START END`
        
        Returns an iterator for the sub-sequence of `SEQUENCE`, identified by the range `[START, END)`.
        
        `START` is the index of the first element to iterate over. `0` indicates the first element of the sequence.
        
        `END` is the index of the element at which to terminate the iteration, i.e. 1 + the index of the last element to be iterated over. `NIL` indicates the end of the sequence.
    
    2.  MAKE-REVERSE-ITERATOR
    
        Generic Function: `MAKE-REVERSE-ITERATOR SEQUENCE START END`
        
        Returns an iterator for the sub-sequence of `SEQUENCE`, identified by the range `[START, END)`, in which the elements are iterated over in reverse order.
        
        Even though the elements are iterated over in reverse order, `START` and `END` are still relative to the start of the sequence, as in `MAKE-ITERATOR`.
        
        `START` is the index of the last element to visit.
        
        `END` is the index of the element following the first element to be iterated over.
    
    3.  ITERATOR
    
        Function: `ITERATOR SEQUENCE &KEY (START 0) END FROM-END`
        
        Returns an iterator for the sub-sequence of `SEQUENCE` identified by the range `[START, END)`.
        
        `START` (defaults to `0` - the start of the sequence) and `END` (defaults to `NIL` - the end of the sequence) are the start and end indices of the sub-sequence to iterated over (see [MAKE-ITERATOR](#make-iterator) and [MAKE-REVERSE-ITERATOR](#make-reverse-iterator) for more information).
        
        If `FROM-END` is true a reverse iterator is created (by `MAKE-REVERSE-ITERATOR`) otherwise a normal iterator is created (by `MAKE-ITERATOR`).

3.  Mandatory Functions

    These functions have to be implemented for all user-defined iterators.
    
    1.  AT
    
        Generic Function: `AT ITERATOR`
        
        Returns the value of the element at the current position of the iterator `ITERATOR`.
        
        The effects of calling this method, after the iterator has reached the end of the subsequence are unspecified.
    
    2.  ENDP
    
        Generic Function: `ENDP ITERATOR`
        
        Returns true if the iterator is at the end of the subsequence, false otherwise.
        
        The end of the subsequence is defined as the position past the last element of the subsequence, that is the position of the iterating after advancing it (by `ADVANCE`) from the position of the last element.
        
        If the subsequence is empty `ENDP` should immediately return true.
        
        **Note:** The default `T` method calls `CL:ENDP` since this function shadows the `CL:ENDP` function.
    
    3.  ADVANCE
    
        Generic Function: `ADVANCE ITERATOR`
        
        Advances the iterator to the next element in the subsequence. After this method is called, subsequent calls to `AT` should return the next element in the sequence or if the last element has already been iterated over, `ENDP` should return true.

4.  Optional Functions

    Implementing the following functions for user-defined iterators is optional either because, a default method is provided which is implemented using the mandatory functions, or the function is only used by a selected few sequence operations.
    
    1.  START
    
        Generic Function: `START ITERATOR`
        
        Returns the element at the current position of the iterator, if the iterator is not at the end of the sequence, otherwise returns `NIL`.
        
        The default method first checks whether the end of the iterator has been reached, using `ENDP`, and if not returns the current element using `AT`.
        
        The default method is equivalent to the following:
        
        ```lisp
        (unless (endp iterator)
          (at iterator))
        ```
    
    2.  (SETF AT)
    
        Generic Function: `(SETF AT) VALUE ITERATOR`
        
        Sets the value of the element at the position, in the sequence, specified by the iterator.
        
        The effects of calling this function when, the iterator is past the end of the subsequence are unspecified.
        
        Implementing this function is only mandatory if destructive sequence operations will be used.
    
    3.  ADVANCE-N
    
        Generic Function: `ADVANCE-N ITERATOR N`
        
        Advances the iterator by `N` elements. This position should be equivalent to the positioned obtained by calling `ADVANCE` `N` times.
        
        The default method simply calls `ADVANCE`, on `ITERATOR`, `N` times.

5.  Macros

    1.  DOITERS
    
        Macro: `DOITERS (&REST ITERS) &BODY BODY`
        
        Iterates over one or more sequences with the sequence iterators bound to variables.
        
        Each element of ITERS is a list of the form `(IT-VAR SEQUENCE . ARGS)`, where `IT-VAR` is the variable to which the iterator is bound, `SEQUENCE` is the sequence which will be iterated over and `ARGS` are the remaining arguments passed to the [ITERATOR](#iterator-func) function.
        
        The bindings to the `IT-VARS`'s are visible to the forms in `BODY`, which are executed once for each element in the sequence. After each iteration the sequence iterators are `ADVANCE`'d. The loop terminates when the end of a sequence is reached.
    
    2.  DOITER
    
        Macro: `DOITER (ITER &REST ARGS) &BODY BODY`
        
        The is the same as [DOITERS](#doiters) except only a single sequence is iterated over.
    
    3.  DOSEQ
    
        Macro: `DOSEQ (ELEMENT SEQUENCE &REST ARGS) &BODY BODY`
        
        Iterates over the elements of `SEQUENCE`. `ARGS` are the remaining arguments passed to the [ITERATOR](#iterator-func) function.
        
        The forms in `BODY` are executed once for each element, with the value of the element bound to `ELEMENT`.


<a id="collector"></a>

### Collector

The collector interface is a generic interface for accumulating items in a sequence/container.

The collector interface is implemented for lists, vectors and `HASH-MAPS`.

1.  MAKE-COLLECTOR

    Generic Function: `MAKE-COLLECTOR SEQUENCE &KEY FRONT`
    
    Returns a collector for accumulating items onto the back of the sequence `SEQUENCE`. If `:FRONT` is provided and is true, the items are accumulated to the front of the sequence rather than back.
    
    The collector may destructively modify `SEQUENCE` however it is not mandatory and may accumulate items into a copy of `SEQUENCE` instead.

2.  ACCUMULATE

    Generic Function: `ACCUMULATE COLLECTOR ITEM`
    
    Accumulates `ITEM` into the sequence associated with the collector `COLLECTOR`.

3.  COLLECTOR-SEQUENCE

    Generic Function: `COLLECTOR-SEQUENCE COLLECTOR`
    
    Returns the underlying sequence associated with the collector `COLLECTOR`. The sequence should contain all items accumulated up to the call to this function.
    
    No items should be accumulated, by `ACCUMULATE` or `EXTEND`, after this function is called.
    
    The sequence returned might not be the same object passed to `MAKE-COLLECTOR`.

4.  EXTEND

    Generic Function: `EXTEND COLLECTOR SEQUENCE`
    
    Accumulates all elements of the sequence `SEQUENCE` into the sequence associated with the collector `COLLECTOR`.
    
    If `SEQUENCE` is an iterator all elements up-to the end of the iterator (till `ENDP` returns true) should be accumulated.
    
    Implementing this method is optional as default methods are provided for iterators and sequences, which simply accumulate each element one by one using `ACCUMULATE`.
    
    Methods:
    
    -   `T ITERATOR`
        
        Accumulates all elements returned by the iterator `SEQUENCE` (till `(ENDP SEQUENCE)` returns true), into the sequence associated with the collector. The elements are accumulated one by one using `ACCUMULATE`.
        
        The iterator is copied thus the position of the iterator passed as an argument is not modified.
    
    -   `T T`
        
        Accumulates all elements of `SEQUENCE`, into the sequence associated with the collector. The elements are accumulated one by one using `ACCUMULATE`.
        
        The sequence iteration is done using the iterator interface.


<a id="orgf85810d"></a>

### Sequences

1.  Creation

    The following functions are for creating a sequence into which items will be accumulated using the collector interface.
    
    1.  CLEARED
    
        Generic Function: `CLEARED SEQUENCE &KEY &ALLOW-OTHER-KEYS`
        
        Returns a new empty sequence, of the same type and with the same properties as `SEQUENCE`, suitable for accumulating items into it using the collector interface.
        
        Individual methods may accept keyword parameters which specify certain options of the sequence which is to be created.
        
        Methods:
        
        -   `LIST`
            
            Returns `NIL`.
        
        -   `VECTOR`
            
            Returns an adjustable vector of the same length as `SEQUENCE`, with the fill-pointer set to `0`.
            
            If the `:KEEP-ELEMENT-TYPE` argument is provided and is true, the element type of the new vector is the same as the element type of `SEQUENCE`.
    
    2.  MAKE-SEQUENCE-OF-TYPE
    
        Generic Function: `MAKE-SEQUENCE-OF-TYPE TYPE ARGS`
        
        Returns a new empty sequence of type `TYPE`. `ARGS` are the type arguments, if any.
        
        The default method creates a built-in sequence of the same type as that returned by:
        
        ```lisp
        (make-sequence (cons type args) 0)
        ```
    
    3.  SEQUENCE-OF-TYPE
    
        Function: `SEQUENCE-OF-TYPE TYPE`
        
        Creates a new sequence of type `TYPE`, using `MAKE-SEQUENCE-OF-TYPE`.
        
        If `TYPE` is a list the `CAR` of the list is passed as the first argument, to `MAKE-SEQUENCE-OF-TYPE`, and the `CDR` is passed as the second argument. Otherwise, if `TYPE` is not a list, it is passed as the first argument and `NIL` is passed as the second argument.

2.  Sequence Elements

    1.  ELT
    
        Generic Function: `ELT SEQUENCE INDEX`
        
        Returns the element at position `INDEX` in the sequence `SEQUENCE`.
        
        Methods:
        
        -   `SEQUENCE T` and `VECTOR T`
            
            Returns `(CL:ELT SEQUENCE INDEX)`.
        
        -   `ARRAY T`
            
            Multi-dimensional Arrays.
            
            Returns `(ROW-MAJOR-AREF SEQUENCE INDEX)`.
        
        -   `T T`
            
            Creates an iterator for `SEQUENCE`, with start position `INDEX`, and returns the first element returned by the iterator.
    
    2.  (SETF ELT)
    
        Generic Function: `(SETF ELT) VALUE SEQUENCE INDEX`
        
        Sets the value of the element at position `INDEX` in the sequence `SEQUENCE`.
        
        Methods:
        
        -   `T SEQUENCE T` and `T VECTOR T`
            
            Returns `(SETF (CL:ELT SEQUENCE INDEX) VALUE)`.
        
        -   `T ARRAY T`
            
            Multi-dimensional Arrays.
            
            Returns `(SETF (ROW-MAJOR-AREF SEQUENCE INDEX) VALUE)`
        
        -   `T T T`
            
            Creates an iterator for `SEQUENCE`, with start position `INDEX`, and sets the value of the element at the starting position of the iterator.
    
    3.  FIRST
    
        Generic Function: `FIRST SEQUENCE`
        
        Returns the first element in the sequence `SEQUENCE`.
        
        Implemented for lists, vectors and multi-dimensional arrays. For multi-dimensional arrays, the first element is obtained by `ROW-MAJOR-AREF`.
        
        The default method is implemented using `GENERIC-CL:ELT`, i.e. is equivalent to:
        
        ```lisp
        (elt sequence index)
        ```
    
    4.  LAST
    
        Generic Function: `LAST SEQUENCE &OPTIONAL (N 0)`
        
        Returns the `N`'th element from the last element of the sequence `SEQUENCE`. `N` defaults to `0` which indicates the last element. `1` indicates the second to last element, `2` the third to last and so on.
        
        Implemented for lists, vectors and multi-dimensional arrays. For multi-dimensional arrays, the last element is obtained by:
        
        ```lisp
        (row-major-aref sequence (- (array-total-size array) 1 n))
        ```
        
        The default method is implemented using `GENERIC-CL:ELT`, i.e. is equivalent to:
        
        ```lisp
        (elt sequence (- (length sequence) 1 n))
        ```
        
        **Note:** The behaviour of this function differs from `CL:LAST` when called on lists, it returns the last element rather than the last `CONS` cell. The [LASTCDR](#lastcdr) function performs the same function as `CL:LAST`.
    
    5.  LASTCDR
    
        Function: `LASTCDR LIST &OPTIONAL (N 1)`
        
        This function is equivalent to `CL:LAST` list function.
        
        Returns the `CDR` of the `N`'th `CONS` cell from the end of the list.
    
    6.  ERASE
    
        Generic Function: `ERASE SEQUENCE INDEX`
        
        Removes the element at index `INDEX` from the sequence `SEQUENCE`.
        
        Destructively modifies `SEQUENCE`.
        
        Methods:
        
        -   `VECTOR T`
            
            Shifts the elements following `INDEX` one elements towards the front of the vector and shrinks the vector by one element.
            
            **Note:** Signals a `TYPE-ERROR` if the vector is not adjustable.
        
        **Note:** This method is not implemented for lists as remove the first element of a list cannot be implemented (efficiently) as a side effect alone.

3.  Length

    1.  LENGTH
    
        Generic Function: `LENGTH SEQUENCE`
        
        Returns the number of elements in the sequence `SEQUENCE`. If `SEQUENCE` is an iterator, returns the number of remaining elements to be iterated over.
        
        This function is implemented for all Common Lisp sequences, returning the length of the sequence (by `CL:LENGTH`), multi-dimensional arrays, returning the total number of elements in the array (by `ARRAY-TOTAL-SIZE`), and hash maps/hash tables, returning the total number of elements in the map/table.
        
        The following default methods are implemented:
        
        -   `ITERATOR`
            
            Returns the number of elements between the iterator's current position (inclusive) and the end of the iterator's subsequence.
            
            This is implemented by advancing the iterator (by `ADVANCE`) till `ENDP` returns true, thus is a linear `O(n)`, in time, operation.
            
            More efficient specialized methods are provided for iterators to sequences for which the size is known.
        
        -   `T`
            
            Returns the length of the generic sequence by creating an iterator to the sequence and calling the `ITERATOR` specialized method. Thus this is a linear `O(n)`, in time, operation unless a more efficient method, which is specialized on the sequence's iterator type, is implemented.

4.  Subsequences

    1.  SUBSEQ
    
        Generic Function: `SUBSEQ SEQUENCE START &OPTIONAL END`
        
        Returns a new sequence that contains the elements of `SEQUENCE` at the positions in the range `[START, END)`. If `SEQUENCE` is an iterator, an iterator for the sub-sequence relative to the current position of the iterator is returned.
        
        `START` is the index of the first element of the subsequence, with `0` indicating the start of the sequence. if `SEQUENCE` is an iterator, `START` is the number of times the iterator should be `ADVANCE`'d to reach the first element of the subsequence.
        
        `END` is the index of the element following the last element of the subsequence. `NIL` (the default) indicates the end of the sequence. If `SEQUENCE` is an iterator, `END` is the number of times the iterator should be `ADVANCE`'d till the end position is reached.
        
        Methods:
        
        -   `SEQUENCE T`
            
            Returns the subsequence using `CL:SUBSEQ`.
        
        -   `ITERATOR T`
            
            Returns a subsequence iterator which wraps a copy of the original iterator.
        
        -   `T T`
            
            Returns the subsequence of the generic sequence. This requires that the `CLEARED` method, the iterator interface and collector interface are implemented for the generic sequence type.
    
    2.  (SETF SUBSEQ)
    
        Generic Function: `(SETF SUBSEQ) NEW-SEQUENCE SEQUENCE START &OPTIONAL END`
        
        Replaces the elements of `SEQUENCE` at the positions in the range `[START, END)`, with the elements of `NEW-SEQUENCE`. The shorter length of `NEW-SEQUENCE` and the number of elements between `START` and `END` determines how many elements of `SEQUENCE` are actually modified.
        
        See [SUBSEQ](#subseq) for more details of how the `START` and `END` arguments are interpreted.
        
        Methods:
        
        -   `SEQEUNCE SEQUENCE T`
            
            Sets the elements of the subsequence using `(SETF CL:SUBSEQ)`.
        
        -   `T T T`
            
            Sets the elements of the generic sequence using the iterator interface, which should be implemented for both the types of `SEQUENCE` and `NEW-SEQUENCE`. This method requires that the `(SETF AT)` method is implemented for the iterator type of `SEQUENCE`.

5.  Sequence Operations

    Generic function wrappers, which are identical in behavior to their counterparts in the `COMMON-LISP` package, are provided for the following sequence operations:
    
    -   `FILL`
    -   `REPLACE`
    -   `REDUCE`
    -   `COUNT`
    -   `COUNT-IF`
    -   `COUNT-IF-NOT`
    -   `FIND`
    -   `FIND-IF`
    -   `FIND-IF-NOT`
    -   `POSITION`
    -   `POSITION-IF`
    -   `POSITION-IF-NOT`
    -   `SEARCH`
    -   `MISMATCH`
    -   `REVERSE`
    -   `NREVERSE`
    -   `SUBSTITUTE`
    -   `NSUBSTITUTE`
    -   `SUBSTITUTE-IF`
    -   `NSUBSTITUTE-IF`
    -   `SUBSTITUTE-IF-NOT`
    -   `NSUBSTITUTE-IF-NOT`
    -   `REMOVE`
    -   `DELETE`
    -   `REMOVE-IF`
    -   `DELETE-IF`
    -   `REMOVE-IF-NOT`
    -   `DELETE-IF-NOT`
    -   `REMOVE-DUPLICATES`
    -   `DELETE-DUPLICATES`
    
    Two methods are implemented, for all functions, which are specialized on the following types:
    
    -   `CL:SEQUENCE`
        
        Simply calls the corresponding function in the `COMMON-LISP` package.
    
    -   `T`
        
        Implements the sequence operation for generic sequences using the iterator interface.
        
        The non-destructive functions only require that the mandatory iterator functions, the collector interface and [CLEARED](#cleared) method are implemented for the sequence's type.
        
        The destructive versions may additionally require that the optional [(SETF AT)](#setf-at) method is implemented as well.
    
    The default value of the `:TEST` keyword arguments is `GENERIC-CL:EQUALP`, this should be the default value when implementing methods for user-defined sequence types. The `:TEST-NOT` keyword arguments have been removed.
    
    The following functions are identical in behavior to their `CL` counterparts, however are re-implemented using the iterator interface. Unlike the functions in the previous list, these are not generic functions since they take an arbitrary number of sequences as arguments.
    
    -   `EVERY`
    -   `SOME`
    -   `NOTEVERY`
    -   `NOTANY`
    
    The following functions either have no `CL` counterparts or differ slightly in behavior from their `CL` counterparts:
    
    1.  MERGE
    
        Generic Function: `MERGE SEQUENCE1 SEQUENCE2 PREDICATE &KEY`
        
        Returns a new sequence, of the same type as `SEQUENCE1`, containing the elements of `SEQUENCE1` and `SEQUENCE2`. The elements are ordered according to the function `PREDICATE`.
        
        Unlike `CL:MERGE` this function is non-destructive.
    
    2.  NMERGE
    
        Generic Function: `MERGE SEQUENCE1 SEQUENCE2 PREDICATE &KEY`
        
        Same as `MERGE` however is permitted to destructively modify either `SEQUENCE1` or `SEQUENCE2`.
    
    3.  SORT
    
        Generic Function: `SORT SEQUENCE &KEY TEST KEY`
        
        Returns a new sequence of the same type as `SEQUENCE`, with the same elements sorted according to the order determined by the function `TEST`. `TEST` is `GENERIC-CL:LESSP` by default.
        
        Unlike `CL:SORT` this function is non-destructive.
        
        For the sort algorithm to be efficient, efficient [ADVANCE-N,](#advance-n) [SUBSEQ](#subseq) and [LENGTH](#length) methods should be implemented for the iterator type of `SEQUENCE`'s.
    
    4.  STABLE-SORT
    
        Generic Function: `STABLE-SORT SEQUENCE &KEY TEST KEY`
        
        Same as `SORT` however the sort operation is guaranteed to be stable. `TEST`. `TEST` is `GENERIC-CL:LESSP` by default.
        
        Unlike `CL:STABLE-SORT` this function is non-destructive.
        
        For the sort algorithm to be efficient, efficient [ADVANCE-N,](#advance-n) [SUBSEQ](#subseq) and [LENGTH](#length) methods should be implemented for the iterator type of `SEQUENCE`'s.
    
    5.  NSORT
    
        Generic Function: `NSORT SEQUENCE &KEY TEST KEY`
        
        Same as `SORT` however is permitted to destructively modify `SEQUENCE`.
    
    6.  STABLE-NSORT
    
        Generic Function: `STABLE-NSORT SEQUENCE &KEY TEST KEY`
        
        Same as `STABLE-SORT` however is permitted to destructively modify `SEQUENCE`.
    
    7.  CONCATENATE
    
        Function: `CONCATENATE SEQUENCE &REST SEQUENCES`
        
        Returns a new sequence, of the same type as `SEQUENCE`, containing all the elements of `SEQUENCE` and of each sequence in `SEQUENCES`, in the order they are supplied.
        
        Unlike `CL:CONCATENATE` does not take a result type argument.
    
    8.  NCONCATENATE
    
        Function: `NCONCATENATE RESULT &REST SEQUENCES`
        
        Destructively concatenates each sequence in `SEQUENCES` to the sequence `RESULT`.
    
    9.  MAP
    
        Function: `MAP FUNCTION SEQUENCE &REST SEQUENCES`
        
        Creates a new sequence, of the same type as `SEQUENCE` (by `CLEARED`), containing the result of applying `FUNCTION` to each element of SEQUENCE and each element of each `SEQUENCE` in `SEQUENCES`.
        
        This is equivalent (in behavior) to the `CL:MAP` function except the resulting sequence is always of the same type as the first sequence passed as an argument, rather than being determined by a type argument.
    
    10. NMAP
    
        Function: `NMAP RESULT FUNCTION &REST SEQUENCES`
        
        Destructively replaces each element of `RESULT` with the result of applying `FUNCTION` to each element of `RESULT` and each element of each sequence in SEQUENCES.
        
        This function is similar in behavior to `CL:MAP-INTO` with the exception that if `RESULT` is a vector, then `FUNCTION` is only applied on the elements up-to the fill pointer i.e. the fill-pointer is not ignored.
    
    11. MAP-INTO
    
        Function: `MAP-INTO RESULT FUNCTION &REST SEQUENCES`
        
        Applies `FUNCTION` on each element of each sequence in `SEQUENCES` and accumulates the result in RESULT, using the [Collector](#collector) interface.
    
    12. MAP-TO
    
        Function: `MAP-TO TYPE FUNCTION &REST SEQUENCES`
        
        Applies `FUNCTION` to each element of each sequence in `SEQUENCES` and stores the result in a new sequence of type `TYPE` (created using `SEQUENCE-OF-TYPE`). Returns the sequence in which the results of applying function are stored.
        
        This function is equivalent in arguments, and almost equivalent in behavior, to `CL:MAP`. The only difference is that if `TYPE` is a subtype of vector, the vector returned is adjustable with a fill-pointer.
    
    13. FOREACH
    
        Function: `FOREACH &REST SEQUENCES`
        
        Applies `FUNCTION` on each element of each sequence in `SEQUENCES`.
        
        Returns `NIL`.


<a id="orga1dad3f"></a>

### Generic Hash-Tables

This interface provides a hash-table data structure with the generic function `HASH` as the hash function and the generic function `GENERIC-CL:EQUALP` as the key comparison function. This allows the hash-tables to utilize keys of user-defined types, whereas the keys of standard hash tables are limited to numbers, characters, lists and strings.

The generic hash-tables are implemented using [CL-CUSTOM-HASH-TABLE](https://github.com/metawilm/cl-custom-hash-table). If the Common Lisp implementation supports creating hash-tables with user-defined hash and comparison functions, standard hash-tables are used. However if the implementation does not support, user-defined has and comparison functions, a fallback solution is used, which is a custom hash-table implementation on top of standard hash-tables. The `HASH-MAP` structure wraps the custom hash-table which allows methods methods to be specialized on a single type `HASH-MAP` regardless of whether standard or custom hash-tables are used. Otherwise two identical methods would have to be implemented, one specializing on standard hash-tables and one specializing on custom hash-tables. More identical methods would have to be implemented if the method has hash-table specializers for more than one arguments, leading to a combinatorial explosion.

The functions in this interface are specialized on the `HASH-MAP` type, due to the issue described above, thus use this type, created with `MAKE-HASH-MAP`, rather than built-in hash-tables. If a hash-table is obtained from an external source, use `HASH-MAP` or `ENSURE-HASH-MAP` to convert it to a `HASH-MAP`.

1.  HASH-MAP

    Structure: `HASH-MAP` with slots: `TABLE`
    
    Function: `HASH-MAP TABLE`
    
    The `HASH-MAP` structure wraps a standard `HASH-TABLE` or `CUSTOM-HASH-TABLE`. The `TABLE` slot, accessed with `HASH-MAP-TABLE`, stores the underlying hash-table.
    
    The `HASH-MAP` function creates a hash-map wrapping `TABLE`.
    
    1.  Implemented Interfaces
    
        The iterator interface is implemented for `HASH-MAP`'s. Each element returned by the iterator is a `CONS`' with the key in the `CAR` and the corresponding value in the `CDR`. The order in which the entries are iterated over is unspecified. Likewise it is unspecified which entries will be iterated over if `START` or `END` is provided, the only guarantee being that `END - START` entries are iterated over. The reverse iterator iterates over the entries in the same order as the normal iterator due to the order of iteration being unspecified.
        
        The [(SETF AT)](#setf-at) method for the `HASH-MAP` iterator sets the value corresponding to the key of the current entry, being iterated over, to the value passed as the argument to `SETF`.
        
        The collector interface is implemented for `HASH-MAP`'s. The `ACCUMULATE` method expects a `CONS` where the `CAR` is the key of the entry to create and the `CDR` is the corresponding value.
        
        An `EQUALP` method is implemented for `HASH-MAP`'s which returns true if both maps contain the same number of `ENTRIES` and each key in the first map is present in the second map, with the corresponding value in the first map equal (by `EQUALP`) to the corresponding value in the second map. **Note:** if the two maps have different test functions, `EQUALP` is not necessarily symmetric i.e. `(EQUALP A B)` does not imply `(EQUALP B A)`.
        
        A `COPY` method is implemented for `HASH-MAPS` which by default creates a new map with the same entries as the original map. If `:DEEP T` is provided the values (but not the keys) are copied by `(COPY VALUE :DEEP T)`.

2.  MAKE-HASH-MAP

    Function: `MAKE-HASH-MAP &KEY TEST &ALLOW-OTHER-KEYS`
    
    Creates a `HASH-MAP` wrapping a hash table with test function `TEST`, which defaults to `#'GENERIC-CL:EQUALP`.
    
    If `TEST` is either the symbol or function `GENERIC-CL:EQUALP`, then a generic hash-table with hash function `HASH` and comparison function `GENERIC-CL:EQUALP` is created. Otherwise `TEST` may be any of the standard hash-table test specifiers.
    
    The function accepts all additional arguments (including implementation specific arguments) accepted by `CL:MAKE-HASH-TABLE`.

3.  ENSURE-HASH-MAP

    Function: `ENSURE-HASH-MAP THING`
    
    If `MAP` is a `HASH-MAP` returns it, otherwise if `MAP` is a `HASH-TABLE` or `CUSTOM-HASH-TABLE` returns a `HASH-MAP` which wraps it. Signals an error if `MAP` is not of the aforementioned types.

4.  HASH

    Generic Function: `HASH OBJECT`
    
    Hash function for hash tables with the `GENERIC-CL:EQUALP` test specifier.
    
    Returns a hash code for `OBJECT`, which is a non-negative fixnum. If two objects are equal (under `GENERIC-CL:EQUALP`) then the hash codes, for the two objects, returned by `HASH` should be equal.
    
    The default method calls `CL:SXHASH` which satisfies the constraint that `(CL:EQUAL X Y)` implies `(= (CL:SXHASH X) (CL:SXHASH Y))`.
    
    Currently no specialized method is provided for container/sequence objects such as lists. The default method does not violate the constraint for lists as keys, provided all `EQUALP` methods specialize on the same types for both their arguments, however will likely be inefficient.

5.  GET

    Generic Function: `GET KEY MAP &OPTIONAL DEFAULT`
    
    Returns the value of the entry corresponding to the key `KEY` in the map `MAP`. If the `MAP` does not contain any entry with that key, `DEFAULT` is returned. The second return value is true if an entry with key `KEY` was found in the map, false otherwise.
    
    Methods are provided `HASH-MAP`'s, standard `HASH-TABLE`'s, association lists (`ALISTS`) and property lists `PLISTS`. For `ALISTS` the `EQUALP` key comparison function is used. For `PLISTS` the `EQ` key comparison function is used.

6.  (SETF GET)

    Generic Function: `(SETF GET) VALUE KEY MAP &OPTIONAL DEFAULT`
    
    Sets the value of the entry corresponding to the key `KEY` in the map `MAP`. `DEFAULT` is ignored.
    
    Only a method for `HASH-MAPS` and `HASH-TABLES` is provided.

7.  ERASE Method

    Method: `ERASE (MAP HASH-MAP) KEY`
    
    Removes the entry with key `KEY` from `MAP`.
    
    Returns true if the map contained an entry with key `KEY`.

8.  HASH-MAP-ALIST

    Function: `HASH-MAP-ALIST MAP`
    
    Returns an association list (`ALIST`) containing all the entries in the map `MAP`.

9.  ALIST-HASH-MAP

    Function: `ALIST-HASH-MAP ALIST &REST ARGS`
    
    Returns a `HASH-MAP` containing all entries in the association list `ALIST`. `ARGS` are the additional arguments passed to `MAKE-HASH-MAP`.

10. COERCE Methods

    The following `COERCE` methods are provided for `HASH-MAPS`:
    
    -   `HASH-MAP (EQL 'ALIST)`
        
        Returns an association list (`ALIST`) containing all the entries in the map. Equivalent to `HASH-MAP-ALIST`.
    
    -   `HASH-MAP (EQL 'PLIST)`
        
        Returns a property list (`PLIST`) containing all the entries in the map.


<a id="org5c19cb7"></a>

### Set Operations

The set interface provides generic functions for performing set operations and implementations of those operations for a hash-set data structure.

Generic function wrappers are provided over the following Common Lisp set operation functions:

-   `SUBSETP`
-   `ADJOIN`
-   `INTERSECTION`
-   `NINTERSECTION`
-   `SET-DIFFERENCE`
-   `NSET-DIFFERENCE`
-   `SET-EXCLUSIVE-OR`
-   `NSET-EXCLUSIVE-OR`
-   `UNION`
-   `NUNION`

For each function, methods specializing on `LISTS`, which simply call the corresponding function in the `CL` package, and `HASH-MAPS` are implemented. Each function accepts all keyword arguments accepted by the corresponding `CL` functions however they are ignored by the `HASH-MAP` methods.

`HASH-MAP`'s may be used as sets, in which case the set elements are stored in the keys. The values of the map's entries are ignored by the set operations, and are unspecified in the sets returned by the set operation functions.

1.  ADJOIN

    Generic Function: `ADJOIN ITEM SET &KEY &ALLOW-OTHER-KEYS`
    
    Returns a new set, of the same type as `SET`, which contains `ITEM` and all element is `SET`.
    
    This function is non-destructive. A new set is always returned even if `SET` is a `HASH-MAP` / `HASH-SET`.
    
    Accepts all keyword arguments accepted by `CL:ADJOIN` however they are ignored by the `HASH-MAP` method.

2.  NADJOIN

    Generic Function: `ADJOIN ITEM SET &KEY &ALLOW-OTHER-KEYS`
    
    Same as `ADJOIN` however is permitted to destructively modify `SET`.
    
    The set returned is `EQ` to `SET` in the case of `SET` being a `HASH-MAP` however is not `EQ` if `SET` is a list, and is not required to be `EQ`. Thus this function should not be relied upon for its side effects.
    
    Implemented for both lists and `HASH-MAPS`.

3.  MEMBERP

    Generic Function: `MEMBERP ITEM SET &KEY &ALLOW-OTHER-KEYS`
    
    Returns true if `ITEM` is an element of the set `SET`.
    
    Implemented for both lists and `HASH-MAPS`. All keyword arguments accepted by `CL:MEMBER` are accepted, however are ignored by the `HASH-MAP` method.

4.  HASH-SET

    Structure: `HASH-SET`
    
    A hash-set is a `HASH-MAP` however it used to indicate that only the keys are important. This allows the `EQUALP` and `COPY` methods, specialized on `HASH-SETS` to be implemented more efficiently, as the keys are not compared/copied, than the methods specialized on `HASH-MAPS`.
    
    The set operations work on both `HASH-MAPS` and `HASH-SETS`.

5.  HASH-TABLE-SET

    Function: `HASH-TABLE-SET TABLE`
    
    Returns a `HASH-SET` structure wrapping the standard `HASH-TABLE` or `CUSTOM-HASH-TABLE` `TABLE`.

6.  HASH-SET

    Function: `HASH-SET &REST ELEMENTS`
    
    Returns a `HASH-SET` with elements `ELEMENTS`.

7.  MAKE-HASH-SET

    Function: `MAKE-HASH-SET &KEY &ALLOW-OTHER-KEYS`
    
    Returns a new empty `HASH-SET`.
    
    Accepts the same keyword arguments as `MAKE-HASH-MAP`. The default `TEST` function is `GENERIC-CL:EQUALP`.


<a id="org56f8601"></a>

### Math Functions

Generic function wrappers are provided over a number of math functions. Methods specialized on `NUMBER` are provided, which simply call the corresponding functions in the `CL` package. The idea of this interface is to allow the mathematical functions to be extended to vectors and matrices. This interface might not used as often as the previous interfaces, thus is contained in a separate package `GENERIC-MATH-CL` which exports all symbols exported by `GENERIC-CL` and shadows the math functions.

Generic function wrappers are provided for the following functions:

-   `SIN`
-   `COS`
-   `TAN`
-   `ASIN`
-   `ACOS`
-   `ATAN`
-   `SINH`
-   `COSH`
-   `TANH`
-   `ASINH`
-   `ACOSH`
-   `ATANH`
-   `EXP`
-   `EXPT`
-   `LOG`
-   `SQRT`
-   `ISQRT`
-   `REALPART`
-   `IMAGPART`
-   `CIS`
-   `CONJUGATE`
-   `PHASE`
-   `NUMERATOR`
-   `DENOMINATOR`
-   `RATIONAL`
-   `RATIONALIZE`


<a id="org2af4923"></a>

## Optimization

There is an overhead associated with generic functions. Code making use of the generic function interface will be slower than code which calls the `CL` functions directly, due to the cost of dynamic method dispatch. For most cases this will not result in a noticeable decrease in performance, however for those cases where it does there is an optimization.

This library is built on top of [STATIC-DISPATCH](https://github.com/alex-gutev/static-dispatch), which is a library that allows generic-function dispatch to be performed statically, at compile-time, rather than dynamically, at runtime. The library allows a call to a generic function to be replaced with the body of the appropriate method, which is selected based on the type declarations of its arguments.

For a generic function call to be inlined, the generic function has to be declared inline (either locally or globally), and the arguments must either have type declarations (if they are variables), or be surrounded in a `THE` form.

Example:

```lisp
(let ((x 1))
  (declare (inline equalp)
	   (type number x))

  (equalp x (the number (+ 3 4))))
```

This will result in the call to the `EQUALP` function being replaced with the body of the `NUMBER NUMBER` method.

The n-argument equality and comparison functions also have associated compiler-macros which replace the calls to the n-argument functions with multiple inline calls to the binary functions, e.g. `(= 1 2 3)` is replaced with `(and (equalp 1 2) (equalp 1 3))`.

Thus the following should also result in the `EQUALP` function calls being statically dispatched, though this has not yet been tested:

```lisp
(let ((x 1))
  (declare (inline equalp)
	   (type number x))

  (= x (the number (+ 3 4))))
```

**Note:** STATIC-DISPATCH requires the ability to extract `TYPE` and `INLINE` declarations from implementation specific environment objects. This is provided by the [CL-ENVIRONMENTS](https://github.com/alex-gutev/cl-environments) library however in order for it to work on all supported implementations, the `ENABLE-HOOK` function (exported by `GENERIC-CL`) has to be called at some point before the generic function call is compiled.

Visit [STATIC-DISPATCH](https://github.com/alex-gutev/static-dispatch) and [CL-ENVIRONMENTS](https://github.com/alex-gutev/cl-environments) for more information about these optimizations and the current limitations.