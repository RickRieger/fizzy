# Fizzy — Single-Number FizzBuzz

[![Tests](https://github.com/RickRieger/fizzy/actions/workflows/tests.yml/badge.svg)](https://github.com/RickRieger/fizzy/actions/workflows/tests.yml)

A JavaScript exercise from the **Code Immersives** web development bootcamp in New York City (February 2021). It's a one-number version of the classic [FizzBuzz](https://en.wikipedia.org/wiki/Fizz_buzz) programming challenge: given a number, return `'Fizz'` if it's divisible by 3, `'Buzz'` if divisible by 5, `'FizzBuzz'` if divisible by both, and otherwise the number itself. [Jest](https://jestjs.io/) tests check each case. This repo is my fork of the assignment. My solution is in [`main.js`](main.js).

**Skills practiced:** the modulus operator (`%`) to test divisibility, `if`/`else if`/`else` chains, and why the **order** of conditions matters.

**Built with:** JavaScript (Node.js) and Jest 24. There's no web page; the badge above shows whether the tests currently pass on GitHub.

---

## Contents

- [My solution](#my-solution)
- [How it works](#how-it-works)
- [Run the tests](#run-the-tests)
- [How it's verified](#how-its-verified)
- [Cleanup log (November 2026)](#cleanup-log-november-2026)
- [Original assignment (from the instructor)](#original-assignment-from-the-instructor)

---

## My solution

Written on **2021-02-22** (commit *"too easy"*). All tests passed. *(Shown as it is after the 2026 cleanup; see the log below for a hidden structural problem that was fixed.)*

```js
function fizzy(num) {
  // Check "both" first: once that case has returned, the next checks don't
  // need to rule it out again.
  if (num % 3 === 0 && num % 5 === 0) {
    return 'FizzBuzz';
  } else if (num % 3 === 0) {
    return 'Fizz';
  } else if (num % 5 === 0) {
    return 'Buzz';
  } else {
    return num;
  }
}
```

## How it works

**Divisible means "remainder 0".** `num % 3` is the remainder after dividing by 3, so `num % 3 === 0` means "divisible by 3". For example, `9 % 3` is `0`, while `10 % 3` is `1`.

| Input | `% 3` | `% 5` | Result |
|---|---|---|---|
| `15` | 0 | 0 | `'FizzBuzz'` |
| `6` | 0 | 1 | `'Fizz'` |
| `20` | 2 | 0 | `'Buzz'` |
| `31` | 1 | 1 | `31` |

**Order matters.** 15 is divisible by 3, so if the `'Fizz'` check came first, 15 would wrongly get `'Fizz'`. Checking "both" first means that by the time JavaScript reaches the `'Fizz'` line, the number is known *not* to be divisible by both, so `num % 3 === 0` alone is enough.

It also handles inputs the tests don't use: `0` gives `'FizzBuzz'` (0 is divisible by everything), `-15` gives `'FizzBuzz'`, and `7.5` comes back unchanged.

## Run the tests

You need [Node.js](https://nodejs.org/).

```bash
git clone https://github.com/RickRieger/fizzy.git
cd fizzy
npm ci
npm run test:ci   # runs all tests once and exits
npm test          # watch mode, as in the original assignment: re-runs on every save
```

Expected result: `Tests: 4 passed, 4 total` (each test checks three numbers).

## How it's verified

[`.github/workflows/tests.yml`](.github/workflows/tests.yml) runs `npm run test:ci` on GitHub Actions after every push, and the badge at the top shows the result. This repo has no web page, so instead of a live site, a passing test run is how it shows it works.

---

## Cleanup log (November 2026)

All changes below were made on **2026-11-23** on the branch `cleanup-and-ci`, checked by running the tests, and merged into `master`. Each is its own commit. Nothing below the assignment's "internal use only" line in `main.js` was edited, and the tests are unchanged.

### 1. Fixed an `else` that had a condition, and added braces

**Why:** the last branch was written like this:

```js
// Before
  else(num % 3 != 0 && num % 5 != 0)
  return num;
```

That looks like "else, if neither, return the number", but **`else` can't take a condition**. Only `if` and `else if` can. JavaScript read it as:

```js
  else (num % 3 != 0 && num % 5 != 0);   // the else body: a check that's computed and thrown away
  return num;                            // a separate line AFTER the whole if-chain
```

It still returned the right answer, but only by accident: `return num;` is reached whenever none of the earlier branches returned. If someone later added another line inside what *looks* like the `else`, it wouldn't behave the way it reads. With no braces on any branch, it was hard to see what belonged where.

```js
// After
  } else {
    return num;
  }
```

Every branch now has braces `{ }`, so each body is explicit.

### 2. Simplified the conditions

**Why:** because the "FizzBuzz" case is checked and returned first, the `'Fizz'` branch doesn't need to also check "and not divisible by 5", and the same goes for `'Buzz'`. Removing those halves also removed the two loose `!=` comparisons. (`!==` is the strict version that the rest of the code uses.)

```js
// Before (after item 1)
  } else if (num % 3 === 0 && num % 5 != 0) {
    return "Fizz";
  } else if (num % 3 != 0 && num % 5 === 0) {
    return "Buzz";

// After
  } else if (num % 3 === 0) {
    return 'Fizz';
  } else if (num % 5 === 0) {
    return 'Buzz';
```

**Checked:** I ran the original 2021 function and the new one side by side on 608 inputs (every whole number from -300 to 300, plus decimals and 100004). They give **identical results for all of them**.

### 3. Automatic test runs on GitHub

**Why:** `npm test` runs Jest in *watch mode*, which never exits, so an automated check couldn't use it. I added a run-once command and a GitHub Actions workflow.

```jsonc
// Before (package.json)
"scripts": { "test": "jest --watch-all" }

// After
"scripts": {
  "test": "jest --watch-all",   // unchanged: watch mode while working
  "test:ci": "jest --ci"        // new: run once and exit
}
```

### 4. Rewrote this README section

Added everything above. The instructor's original instructions are below, unchanged.

---

## Original assignment (from the instructor)

*The text below is the assignment README as written by the Code Immersives instructor, unchanged.*

### Fizzy

### Introduction

[FizzBuzz](https://en.wikipedia.org/wiki/Fizz_buzz) is a famous programming challenge meant to test your approach to a simple problem. Today we'll be doing a very simple version of that simple problem. No loops (yet), but single-number FizzBuzz is an excellent test of our branching logic abilities! What's more, there are many different approaches to this problem--what path will you take?

### Tools Used

You'll get practice with the following:

- Full-featured functions (parameters and return values).
- `if-else` statements.
- Modulus operations.

### Setup

You'll only write one function this time, but there are various inputs you'll need to handle, so this is a test-driven exercise.

- Fork, clone, and navigate to this repo.
- In a terminal (preferably a full-screen one), run `jest --watch-all`.
- Code away until the results are all green.
