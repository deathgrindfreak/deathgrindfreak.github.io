<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. SICP Chapter 2 Solutions</a>
<ul>
<li><a href="#sec-1-1">1.1. 2.2.3 Sequences as Conventional Interfaces</a>
<ul>
<li><a href="#sec-1-1-1">1.1.1. Sequence Operations</a></li>
<li><a href="#sec-1-1-2">1.1.2. Nested Mappings</a></li>
</ul>
</li>
<li><a href="#sec-1-2">1.2. 2.2.4 Example: A Picture Language</a>
<ul>
<li><a href="#sec-1-2-1">1.2.1. The picture language</a></li>
<li><a href="#sec-1-2-2">1.2.2. Higher-order operations</a></li>
<li><a href="#sec-1-2-3">1.2.3. Frames</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

&#x2014;
layout: post
title: SICP Chapter 2 Solutions
date:  2015-01-24 23:55:32
categories: sicp books
&#x2014;

# SICP Chapter 2 Solutions<a id="sec-1" name="sec-1"></a>

These are my notes for the latter half of Chapter 2.  Since I started
using org mode quite a bit after I began SICP, I'll have to come back
and transcribe the previous sections at a later time (Like that's ever 
going to happen).

## 2.2.3 Sequences as Conventional Interfaces<a id="sec-1-1" name="sec-1-1"></a>

One of the most important concepts in this section is the concept of
the "signal chain" as illustrated in fig. 2.7.  Of course this is
really a visualization of how programs are more conceptually tractable
when they are modularized appropriately (a concept that is hammered on
in the famous ["Why Functional Programming Matters"](http://worrydream.com/refs/Hughes-WhyFunctionalProgrammingMatters.pdf) paper).

### Sequence Operations<a id="sec-1-1-1" name="sec-1-1-1"></a>

As the title suggests, this sub-section takes on the task of breaking
down programs into sub-tasks using the signal chain metaphor.

1.  Exercise 2.33

    The best explanation for how to define functions in terms of folds (or
    just writing folds in the first place) actually came from the
    previously mentioned paper.  All a fold (right fold) does on a list is
    replace every 'cons' in the list with the function that is being
    folded with and the final 'nil' with the initial element.  For
    example, 'fold + 0 '(1 2 3) transforms the list like so:
    
        (cons 1 (cons 2 (cons 3 '()))) -> (+ 1 (+ 2 (+ 3 0)))
    
    In the first expression, we merely want to replace every
    'cons x' with a 'cons (p x)'. Show the resulting expression
    is:
    
        (define (map p sequence)
          (accumulate (lambda (x y) (cons (p x) y) nil sequence))
    
    In the second expression, we want the cons to remain unchanged.
    However, in order to append the lists, the final '() in seq1 must be
    replaced by the entire seq2.  Thus the expression becomes:
    
        (define (append seq1 seq2)
          (accumulate cons seq1 seq1))
    
    Finally, for the last expression, each cons must be replaced with '+',
    each non-nil element with 1 and the final 'nil' with 0.
    
        (define (length sequence)
          (accumulate (lambda (x y) (+ 1 y)) 0 sequence))

2.  Exercise 2.34

    Just as was done in the previous exercise, we need to replace the
    conses in the list in order to perform the fold.  The following should
    do the trick:
    
        (define (horner-eval x coefficient-sequence)
          (accumulate (lambda (this-coeff higher-terms)
                        (+ this-coeff
                           (* x
                              higher-terms)))
                      0
                      coefficient-sequence))

3.  Exercise 2.35

    Using the 'replace cons' metaphor, if we replace all conses that are
    themselves pairs with a recursive call to count-leaves, and all
    non-pairs with '1', it will successfully calculate the number of
    leaves in any tree:
    
        (define (count-leaves t)
          (accumulate + 
                      0
                      (map (lambda (x)
                             (if (pair? x)
                                 (count-leaves x)
                                 1))
                           t)))

4.  Exercise 2.36

    I had to stare at an open buffer for a few minutes until the
    solution for this one hit me.  Since we're accumulating only the first
    element of every subsequence at a time, we just have to map the car
    function on the sequence of sequences, pass that to the normal
    accumulate function, the pass the result of mapping the cdr to the
    sequence of sequences to the accumulate-n function (consing these two
    in the process).  Add, rinse, repeat.
    
        (define (accumulate-n op init seqs)
          (if (null? (car seqs))
              nil
              (cons (accumulate op init (map car seqs))
                    (accumulate-n op init (map cdr seqs)))))

5.  Exercise 2.37

    These aren't too bad if you're familiar with some standard matrix
    operations.
    
    In order to define the matrix-vector product, we need to map the
    vector to every row of the matrix and perform a dot product.
    
        (define (matrix-*-vector m v)
          (map (lambda (w) (dot-product v w)) m))
    
    For the next operation, we need to cons each element in each row of
    the matrix, with elements in other rows in the same column.
    
        (define (transpose mat)
          (accumulate-n cons nil mat))
    
    For the final expression, in order to perform the matrix-matrix
    product, we simply need to transform the matrix to the right, then map
    each row in the left, to a dot product with each row in the now
    transposed matrix to the right.
    
        (define (matrix-*-matrix m n)
          (let ((cols (transpose n)))
            (map (lambda (row-m)
                   (map (lambda (row-n)
                          (dot-product row-m row-n))
                        cols))
                 m)))

6.  Exercise 2.38

    The values for the expressions are:
    
        (fold-right / 1 (list 1 2 3)) => (/ 1 (/ 2 (/ 3 1))) => 3/2
        (fold-left / 1 (list 1 2 3)) => (/ (/ (/ 1 1) 2) 3) => 3/2
        (fold-right list nil (list 1 2 3)) => (list 1 (list 2 (list 3 nil)))
        (fold-left list nil (list 1 2 3)) => (list (list (list 1 nil) 2) 3)
    
    From the expressions above, we see that the types of expressions that
    are invariant over folds tend to be those that are commutative. The
    above example for is actually a bit of a misnomer since 
    
        (fold-left / 2 '(1 2 3 4) => 1/12 != 3/4 <= (fold-right / 2 '(1 2 3 4))
    
    However, using + we can see that:
    
        (fold-left + 1 '(1 2 3 4)) => (+ (+ (+ (+ 1 1) 2) 3) 4) = (+ 4 (+ 3 (+ 2 (+ 1 1)))) <= (fold-right + 1 '(1 2 3 4))
    
    This is of course due to the fact that (+ a b) `= (+ b a). thus for
    all all op such that (op a b) =` (op b a), foldr will produce the same
    result as foldl.

7.  Exercise 2.39

    It immediately comes to mind that we should use some sort of list
    operation (no duh) like cons or list, since we're not reducing, but
    merely manipulating the form of the list.  I generally find right
    folds to be a bit easier to grasp conceptually, since they generally
    don't change the structure of the list, so we'll start there.
    
    In order to reverse the entire list, at each step in the fold we can
    simply reverse the order of the parameters to the lambda function and
    cons them.
    
        (define (reverse sequence)
          (fold-right (lambda (x y) (cons y x)) nil sequence))
    
    We can do a similar thing with the left fold, but since at each step
    the fold is taking the previous result and placing it in the right
    argument of op with the next element of the list into the left
    argument, if we instead use list with the arguments reversed, we gain
    the desired procedure (that's one hell of a run-on sentence).
    
        (define (reverse sequence)
          (fold-left (lambda (x y) (list y x)) nil sequence))

### Nested Mappings<a id="sec-1-1-2" name="sec-1-1-2"></a>

This section covers more on sequence operations, mainly on the subject
of nested mappings and their applications.

1.  Exercise 2.40

    I'm not completely sure why they stuck this one in here, mainly since
    they did the work for us at the very beginning of the sub-section
    (perhaps to further illustrate the power of modular programs).
    
        (define (unique-pairs n)
          (flatmap (lambda (i)
                     (map (lambda (j)
                            (list i j))
                          (enumerate-interval 1 (- i 1))))
                   (enumerate-interval 1 n)))
    
    And prime-pairs becomes:
    
        (define (prime-sum-pairs n)
          (map make-pair-sum
               (filter prime-sum? (unique-pairs n))))

2.  Exercise 2.41

    You can guess the structure of this one pretty easily, though it
    unintuitively (at least for me at first) requires two flatmaps, with
    the last one being the only normal map.
    
        (define (unique-triples n)
          (flatmap (lambda (i)
                     (flatmap (lambda (j)
                                (map (lambda (k)
                                       (list i j k))
                                     (enumerate-interval 1 (- j 1))))
                              (enumerate-interval 1 (- i 1))))
                   (enumerate-interval 1 n)))

3.  Exercise 2.42

    I'm actually pretty stoked about this problem, even though took me an
    embarrassing amount of time to complete. But I completed it without
    help nonetheless. My solution is something, I think, that the authors
    didn't intend (since both my versions of the functions adjoin-position
    and safe? don't actually require the input k, but I included it anyway
    in order to stick with the interface presented in the book). Anyway,
    onto the solution.
    
    I tested a few of the initial solutions by visual inspection alone.
    But upon visiting the [wiki page](http://en.wikipedia.org/wiki/Eight_queens_puzzle) on the subject, I discovered that
    there are only 92 solutions out of the total possible 4,426,165,368
    (the total number of ways to arrange 8 queens on the board). And lo
    and behold, my version yields 92 solutions! This is of course not a
    rigorous proof by any measure of the word, but good enough for me.
    
    I think the canonical way to do this (judging by the k variable in the
    functions) was to generate and keep in memory many kxk boards full of
    whatever null representation the programmer chose to user, and by
    inserting values into the proper rows, the solution is generated.
    However, the way that came to my mind was to simply cons new rows onto
    the boards as they came up. That way, things become a bit simpler
    since there's now no need to deal with any unnecessary null rows in
    the board.
    
    For the empty board representation, we simply need a list of the null
    list.  Thus the following definition works nicely:
    
        (define empty-board nil)
    
    Next, the safe? method needs to check all of the columns and diagonals
    in the previous rows for any intersections (the way the rows are
    generated guarantees that there will be no conflict there).
    
        (define (safe? k positions)
          (define (ones-position row)
            (if (= 1 (car row))
                0
                (+ 1 (ones-position (cdr row)))))
        
          (define (check-column check rest)
            (let ((check-position (ones-position check)))
              (null? (filter (lambda (row)
                               (= (ones-position row)
                                  check-position))
                             rest))))
        
          (define (check-diagonal check rest)
            (let ((check-position (ones-position check)))
              (define (diag-helper rest count)
                (if (null? rest)
                    #t
                    (let ((current-row (car rest)))
                      (if (or (= check-position
                                 (+ (ones-position current-row) count))
                              (= check-position
                                 (- (ones-position current-row) count)))
                          #f
                          (diag-helper (cdr rest) (+ count 1))))))
              (diag-helper rest 1)))
        
          (let ((check-row (car positions))
                (rest (cdr positions)))
            (and (check-column check-row rest)
                 (check-diagonal check-row rest))))
    
    I know of a few ways implement this using built-in functions, but I
    thought I'd try to stick with only those functions that have been
    introduced in the book thus far.
    
    Finally, I had to implement the adjoin-positions function as a
    closure, since it needs the board-size definition from the queens
    function. All adjoin-positions needs to really do is cons new rows
    with every possible queen position onto the existing partially-formed
    boards.
    
        (define (queens board-size)
        
          (define (adjoin-position new-row k rest-of-queens)
            (define (make-row-with-k n k)
              (cond ((<= n 0) '())
                    ((= (- n 1)
                        (- n k)) (cons 1
                                       (make-row-with-k (- n 1)
                                                        (- k 1))))
                    (else (cons 0 
                                (make-row-with-k (- n 1)
                                                 (- k 1))))))
            (cons (make-row-with-k board-size new-row)
                  rest-of-queens))
        
          (define (queen-cols k)  
            (if (= k 0)
                (list empty-board)
                (filter
                 (lambda (positions) (safe? k positions))
                 (flatmap
                  (lambda (rest-of-queens)
                    (map (lambda (new-row)
                           (adjoin-position new-row k rest-of-queens))
                         (enumerate-interval 1 board-size)))
                  (queen-cols (- k 1))))))
          (queen-cols board-size))
    
    I'm sure that there are much shorter ways to solve this problem, but
    I'm pretty happy with the way it performs (it can generate the 10x10
    solution in a second or so), so I'll stand behind it. 

4.  Exercise 2.43

    1.  TODO 

## 2.2.4 Example: A Picture Language<a id="sec-1-2" name="sec-1-2"></a>

### The picture language<a id="sec-1-2-1" name="sec-1-2-1"></a>

This is how abstraction works at its best.  The simply fact that the
only object we have to deal with is the 'painter' shows how powerful
of a concept it can be.

1.  Exercise 2.44

    The up-split is almost identical in form to the right split, with the
    interchange of the beside and below functions. 
    
        (define (up-split painter n)
          (if (= n 0)
              painter
              (let ((smaller (up-split (- n 1))))
                (below painter (beside smaller smaller)))))

### Higher-order operations<a id="sec-1-2-2" name="sec-1-2-2"></a>

1.  Exercise 2.45

        (define (split left-op right-op)
          (lambda (painter n)
            (if (= n 0)
                painter
                (let ((smaller (split (- n 1))))
                  (left-op painter (right-op smaller smaller))))))

### Frames<a id="sec-1-2-3" name="sec-1-2-3"></a>

1.  Exercise 2.46

2.  Exercise 3.47