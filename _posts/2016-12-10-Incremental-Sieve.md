---
layout: post
title:  "Generating Infinitely Many Prime Numbers"
date:   2016-12-10 18:30:10 -0500
categories: programming math
---

I recently started working through some [Project Euler](https://projecteuler.net/) problems, and one that was particularly interesting to me so far was #7:

>By listing the first six prime numbers: 2, 3, 5, 7, 11, and 13, we can see that the 6th prime is 13.  
What is the 10 001st prime number?

The way I learned to generate prime number was by using the Sieve of Eratosthenes, and that can be used to solve this problem. However, while thinking about it I realized that I had never had to generate an unspecified number of primes. I always had a upper bound.

So how do you generate an infinite number of primes? My first idea was just to use a resizing array and stick with the standard Sieve of Eratosthenes. That means that we would have to redo all the work each time we hit the upper bound. We can definitely do better than that. After thinking about it for a bit, I realized that for each prime you find, everything prior in the sieve is useless - just wasting space. Also, we really only need to keep track of the next multiple of each prime, instead of marking all of them until some upper bound. This is nice because we can lazily generate the next composites as we go along.

So anyways, I come to find out that there is a cool method for generating an infinite number of primes called the Incremental Sieve of Eratosthenes. 

Here is my Golang implementation:

{% highlight golang %}

func NextPrime() <-chan int {
    out := make(chan int, 1)

    go func() {
        out <- 2
        composites := make(map[int][]int)
        num := 3

        for {
            if _, ok := composites[num]; !ok {
                out <- num
                composites[num * num] = []int{num}
            } else {
                for _, prime := range composites[num] {
                    next := num + prime
                    for next % 2 == 0 {
                        next += prime
                    }
                    if _, ok := composites[next]; ok {
                        composites[next] = append(composites[next], prime)
                    } else {
                        composites[next] = []int{prime}
                    }
                }
                delete(composites, num)
            }
            num += 2
        }
    }()

    return out
}
{% endhighlight %}