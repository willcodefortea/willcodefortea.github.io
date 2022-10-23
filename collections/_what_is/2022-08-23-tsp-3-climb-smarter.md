---
title: "TSP 3: Finding a better hill"
categories: what-is
layout: single
tags: Algorithm

header:
  overlay_image: /assets/images/what-is/tsp-1/salesperson.jpeg
  overlay_filter: 0.6
excerpt: Hill climbing traps us in a minima / maxima, introducing a diversification step allows to break free, potentially finding better solutions.
---

So far we've introduced the problem, generated a random solution, then applied a hill climbing algorithm to improve the overall solution with the simplest possible move.

And you know what, it does pretty well! Especially considering that the deterministic approach is [non-tractable](https://en.wikipedia.org/wiki/Computational_complexity_theory#tractable_problem), getting a reasonably good solution in a faction of a second isn't too bad.


## Introducing 2-opt

So, what can we do from here? Well, the first thing I'd like to do is introduce a new movement operator, called 2-opt. This is a movement that is designed to undo sections that cross-over themselves, netting an overall reduced distance travelled.

{% include figure image_path="/assets/images/what-is/tsp-3/2-opt.png" alt="Diagram of the impact of a 2-opt movement." caption="2-opt undoing a section of a path that cross over itself, reducing the overall distance travelled." %}

This is a movement that is currently impossible with the single move that we have, as it'd need to occur in two steps, with the first being a worse solution. But what are we actually doing here? If we look at the two paths, we have `a - d - c - b - e` then `a - b - c - d - e`, the change that we actually make is the sub-path `d - c - b` is reversed, but left in the same place.

As code, we might write a new movement that looks like this:

```typescript
const twoOptSwap = <T>(arr: T[], a: number, b: number): T[] => {
  return [
    ...arr.slice(0, a),
    ...arr.slice(a, b).reverse(),
    ...arr.slice(b)
  ];
};

const twoOpt: Move = {
  apply(solution) {
    const currentPath = solution.getPath();
    for (let a = 0; a < solution.cities.length - 1; a++) {
      for (let b = a + 1; b < solution.cities.length; b++) {
        const swappedPath = twoOptSwap(currentPath, a, b);
        const candidateSolution = solution.withNewPath(swappedPath);

        if (candidateSolution.cost() < solution.cost()) {
          return candidateSolution;
        }
      }
    }

    // no better solution was found
    return solution;
  },
};
```

while we change the solution class to expose some handy methods for accessing and setting a new path, guaranteeing immutability.

```typescript
class Solution {
  ...

  withNewPath(cities: City[]): Solution {
    const trip = new Trip(cities);
    return new Solution(this.cities, trip);
  }

  getPath(): City[] {
    return [...this.trip.cities];
  }

  ...
}
```

Lovely!

But you may have noticed that we now have two possible moves that we can apply to our solution, and only one interface to call only one, what gives? Well, now we can introduce a `RandomMover`, which picks and applies one of the moves entirely at random.

```typescript
const MOVES = [moveCity,twoOpt];

const randomMove = () => MOVES[randomInt(MOVES.length)];

const randomMover: Move = {
  apply: (solution: Solution) => {
    return randomMove().apply(solution);
  },
};
```

## I didn't like this hill anyway

So we've introduced a new move that will help get us untangled, but we still have quite a big problem. The search space is _large_, how can we insure that we're not stuck in a local minima / maxima? What if the hill that we're currently trying to climb, is just a bad hill?

{% include figure image_path="/assets/images/what-is/tsp-3/local-hill-climb.png" alt="Chart of solution space demonstrating the impact of climbing a locally bad hill." caption="We can see that despite there being many other solutions nearby, we can never reach them as we're only allowed to go up, not down." %}

We've seen that hill-climbing is really good at intensification, if you have a pretty good starting point then you can be confident that you'll walk away with a better solution. But as we can never be certain that we've climbed the highest peak, what if we looked further afield?

To do so, we introduce the concept of a _neighbourhood search_, rather than just search locally, let's explore a little further away and see what happens. To do this, we need to apply a _diversification_ step.

{% include figure image_path="/assets/images/what-is/tsp-3/diversification.png" alt="Chart of solution space demonstrating the impact of diversification." caption="Despite a diversification step producing a much worse result, twe see that it goes on to produce an overall better solution." %}

So what does this look like? Well, first of all we need a new `Runner`. The current one is pure hill climbing, let's extend it. This diversification is often referred to a "destructive" move, as we're taking something seemingly good and making it worse.

```typescript
type RunnerConfig = {
  construction: Move;
  stoppingCriterion: StoppingCriterion;
  move: Move;
};

type NeighbourHoodConfig = RunnerConfig & {
  staleMoveLimit: number;
};

class Runner {
  run(
    cities: City[] = ATT_48_CITIES,
    config: NeighbourHoodConfig,
    initialSolution?: Solution,
    callback: (solution: Solution) => void = () => {}
  ) {
    const emptySolution = Solution.default(cities);
    let bestOverallSolution =
      initialSolution || config.construction.apply(emptySolution);

    let movesSinceBest = 0;
    let currentBestSolution = bestOverallSolution;

    while (!config.stoppingCriterion.shouldStop()) {
      const candidateSolution = config.move.apply(currentBestSolution);

      if (candidateSolution.cost() < currentBestSolution.cost()) {
        movesSinceBest = 0;
        currentBestSolution = candidateSolution;

        if (currentBestSolution.cost() < bestOverallSolution.cost()) {
          bestOverallSolution = currentBestSolution;
        }

        callback(currentBestSolution);
      }

      if (movesSinceBest++ >= config.staleMoveLimit) {
        currentBestSolution = this.destroySolutionSegment(currentBestSolution);
      }
    }

    return bestOverallSolution;
  }

  destroySolutionSegment(solution: Solution): Solution {
    // destroys up to half of the current solution
    const currentPath = solution.getPath();
    const a = randomInt(Math.round(currentPath.length / 2));
    const b = a + randomInt(Math.round(currentPath.length / 2));
    return solution.withNewPath([
      ...currentPath.slice(0, a),
      ...shuffle(currentPath.slice(a, b)),
      ...currentPath.slice(b),
    ]);
  }
}
```

Neat! And in practice? Well, have a play and see for yourself :)

<div id="tsp-app-root"></div>

<br />

Comparing the hill climbing and the neighbourhood runs, you should find that we more frequently end up with better solutions form the neighbourhood search! That doesn't stop you being lucky with the pure hill climbing however, you could very well just guess the best hill to start on.

So far we've been able to create solutions to a problem that is deterministically not-ideal to solve, all in javascript in a fraction of a second. Neat huh?

<script type="text/javascript">
  window.TSP_CONFIG = {
    randomEnabled: true,
    singleMoveEnabled: true,
    neighbourHoodEnabled: true,
  }
</script>
<script type="module" crossorigin src="/assets/apps/tsp/index.9a908d6e.js"></script>
<script type="module" crossorigin src="/assets/apps/tsp/worker.e18e5bfb.js"></script>
<link rel="stylesheet" href="/assets/apps/tsp/index.7ba47d89.css">