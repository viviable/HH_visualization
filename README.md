# Pareto front of MOEA/D alone on SecRepoBench

Interactive companion to the Pareto-front figure in the appendix of the paper.
Open `index.html` in a browser, or view the hosted page.

The page plots the baseline and all 15 children of one controlled MOEA/D run
(Tchebycheff acceptance, N = 5 subproblems, neighborhood T = 2, 3 generations,
no factored evolving or dynamic orchestration) on the 30-task SecRepoBench
search split. Drag the slider to step through generations; hover a point for its
functionality, security and F&S scores.

## Pages

- [`index.html`](index.html): Pareto front of MOEA/D alone on SecRepoBench (search split), as in the appendix figure.
- [`moead_secrepo_data.html`](moead_secrepo_data.html): SecRepoBench held-out Pareto fronts (IN vs. OUT), generation by generation.
- [`moead_baxbench_data.html`](moead_baxbench_data.html): BaxBench search-set and held-out OUT Pareto fronts, generation by generation.
