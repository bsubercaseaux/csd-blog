+++
# The title of your blogpost. No sub-titles are allowed, nor are line-breaks.
title = "Classical AI for Mathematical Research"
# Date must be written in YYYY-MM-DD format. This should be updated right before the final PR is made.
date = 2025-03-24

[taxonomies]
# Keep any areas that apply, removing ones that don't. Do not add new areas!
areas = ["Artificial Intelligence", "Theory"]
# Tags can be set to a collection of a few keywords specific to your blogpost.
# Consider these similar to keywords specified for a research paper
tags = ["SAT", "discrete geometry", "Erdős-Szekeres problems", "automated mathematics"]

[extra]
author = {name = "Bernardo Subercaseaux", url = "https://bsubercaseaux.github.io/"}
# The committee specification is a list of objects similar to the author.
committee = [
{name = "Harry Q. Bovik", url = "http://www.cs.cmu.edu/~bovik/"},
{name = "Committee Member 2's Full Name", url = "Committee Member 2's page"},
{name = "Committee Member 3's Full Name", url = "Committee Member 3's page"}
]
+++

# The Elephant in the Room: AI \& Mathematics

In the last couple of years a pressing question has started to permeate the mathematical community: *_how will mathematicians' jobs coexist harmoniously with AI as it gets progressively better at mathematics?_* ["A.I. Is Coming for Mathematics, Too"](https://www.nytimes.com/2023/07/02/science/ai-mathematics-machine-learning.html) was the title chosen by the NY Times in their dedicated article of 2024, ["AI Will Become Mathematicians' Co-Pilot"](https://www.nytimes.com/2023/07/02/science/ai-mathematics-machine-learning.html) said the Scientific American, and ["Questions Artificial Intelligence Raises for the Mathematics Profession"](https://www.nytimes.com/2023/07/02/science/ai-mathematics-machine-learning.html) chose the American Mathematical Society.

This time it does not seem to be a matter of sensationalistic journalism; some of the most respected mathematicians in the world are joining the conversation, with voices of concern, excitement, and a wide spectrum of positions in between. Perhaps the most aligned with the spirit of this research is Jordan Ellenberg, number theorist who co-authored FunSearch with DeepMind and stated:

> What’s most exciting to me is modeling new modes of human–machine collaboration,
> I don’t look to use these as a replacement for human mathematicians, but as a force multiplier.

The goal of this post is to show through a concrete case study how SAT solvers, a relatively old AI technology, can also allow for human-machine collaboration in mathematics. Interestingly, the problem we study takes place in two-dimensional geometry, an a priori continuous domain, which makes the participation of SAT solvers relatively surprising. While this is not the first usage of SAT in discrete geometry, its main novelty (based the technical results) is in documenting how SAT solvers and other automated reasoning tools can serve a mathematical assistant that can reveal patterns and ellicit conjectures. Furthermore, at the end of the post I discuss verification: as opposed to recent general-purpose forms of AI, standard automated reasoning tools such as (Max)SAT solvers can provide proofs for their answers, which can be crucial for mathematics.

# The Happy Ending was just the beginning

In 1933, Esther Klein presented the following problem to George Szekeres and Paul Erdős:

> **Problem 1.** If five points lie on a plane so that no three points form a straight line, prove that four of the points will form a convex quadrilateral.

Klein's solution is simple and elegant; almost entirely exposed in **Figure 1**. Note first that there are only three possible cases for the size of the convex hull of five points without collinarities. In case **(a)**, where the convex hull includes all five points, any four of them make a convex quadrilateral. In case **(b)**, where the convex hull has only four points, those four are the sought-after quadrilateral. In case **(c)**, where the convex hull is a triangle, we consider the line \\(L\\) passing through the two points inside the triangle, which we call \\(p_1\\) and \\(p_2\\), and then observe that the pigeonhole principle implies that one side of \\(L\\) must contain two of the three points in the convex hull, which we call \\(p_3\\) and \\(p_4\\); we are done, since the points \\(p_1, p_2, p_3, p_4\\) must form a convex quadrilateral.

<!-- ![Illustration of Klein's proof](./klein_cases.png) -->
<img src="./klein_cases.png" width="700" alt="Illustration of Klein's proof for the existence of a convex quadrilateral among 5 popints in general position.">

Klein's problem kickstarted geometric Ramsey theory, and it was soon afterwards extended by Szekeres and Erdős into the following theorem:

> **Erdős-Szekeres Theorem.** For any positive integer \\(k\\), there is an integer \\(g(k)\\) such that any collection of \\(g(k)\\) points contains either 3 collinear points or a convex \\(k\\)-gon.

As a second consequence of the problem, George Szekeres and Esther Klein married, which led Erdős to jokingly name Klein's problem as the *_Happy Ending problem_*.

Not only has Klein's problem had a long-lasting impact in discrete geometry; her solution contains already an important insight: properties of finite point sets such as convexity do not rely on the specific coordinates of the points, but rather on their relative position and orientations, which are enough to determine the structure of e.g., convex hulls. This insight, as it will clearer later in this post, is what opens the door to logical computation in a domain which may a priori seem continuous.

# The Problem: Minimizing Convex Pentagons

Klein's proof shows \\(g(4) \leq 5\\), and it is then easy to see that indeed \\(g(4) = 5\\). It is not too hard to see that \\(g(5) = 9\\) (see **Figure 2**), but it took until 2006 for \\(g(6) = 17\\) to be proven computationally by Lindsay Peters and George Szekeres. [^szekeres] No other values of \\(g(k)\\) are known, although  Erdős and Szekeres conjectured \\(g(k) = 2^{k-2} + 1\\) for all \\(k \geq 3\\).

<img src="./g_5_8_9.png" width="600" alt="8 points without convex pentagons">
<!-- </center> -->
<!-- ![8 points without convex pentagons](g_5_8_9.png) -->

In 1973, a more quantitative variant was posed by Erdős and Guy:

> More generally, one can ask for the least number of convex \\(k\\)-gons determined by \\(n\\) points in the plane [without three on a line].

Indeed, let us denote by \\(\mu_k(n)\\) the minimum number of convex \\(k\\)-gons on a placement of \\(n\\) points in the plane without any three on a common line. While \\(\mu_4(n)\\) has been heavily studied, its close cousin \\(\mu_5(n)\\) has received significantly less attention, and our goal was to remedy this situation. The following table summarizes our progress in computing \\(\mu_5(n)\\), previously known up to \\(n = 11\\), and now solved up to \\(n = 16\\).

| # of points (\\(n\\)) | ≤8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
|-----------------|----|----|----|----|----|----|----|----|----|
| Previously [2]  | 0  | 1  | 2  | 7  | [12, 13] | [20, 34] | [40, 62] | [60, 113] | — |
| Our work        | 0  | 1  | 2  | 7  | 12 | 27 | 42 | 77 | 112 |

<!-- What can we say about $\mu_5(n)$? Well, from $g(5) = 9$ we at least get that $\mu_5(n) = 0$ for $n < 9$, and $\mu_5(9) \geq 1$. We can leverage this fact as follows. If we have for instance $90$ points in the plane, and partition them into $10$ groups of $9$ points, we know that each group must contain at least one convex pentagon, and thus $\mu_5(90) \geq 10$. This folklore "supersaturation" idea can be extended further:

> **Lemma 1**: Let $m$ and $r$ be values such that $\mu_k(m) \geq r$. Then, for every $n \geq m$ we have

$$

\mu_k(n) \geq r \cdot \binom{n}{m}/\binom{n-k}{m-k} = r \binom{n}{k}/\binom{m}{k}.

$$

_Proof_.  -->

<!-- The value of $g(7)$ is conjectured to be $33$, and more in general, Erdős and Szekeres conjectured $g(k) = 2^{k-2} + 1$. In 1961, Erdős and Szekeres proved $g(k) \geq 2^{k-2} + 1$, and to this day, the best upper bound is $g(k) \leq 2^{k + O(\sqrt{k \log k})}$ -->

# Conclusions


This blog is based on joint work with John Mackey, Marijn J.H. Heule and Ruben Martins, which was accepted at CICM'2024, and is publicly available at [https://arxiv.org/abs/2311.03645](https://arxiv.org/abs/2311.03645).

# Bibliography

[^szekeres]: George Szekeres died in 2005 (one hour apart from Esther Klein!), and thus to the best of my knowledge, Peters completed the paper using some previous ideas of Szekeres, which makes unclear whether the Hungarian mathematician got to see the result \\(g(6) = 17\\) before his passing.