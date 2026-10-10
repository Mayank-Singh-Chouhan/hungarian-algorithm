# Hungarian Algorithm Visualizer

## Overview

This project is an interactive demonstration of the **Hungarian algorithm**.

The Hungarian algorithm solves the **assignment problem**:

> Assign workers to jobs so that every worker gets one job and the total cost is as small as possible.

It uses:

- HTML
- JavaScript
- Tailwind CSS (CDN)
- Font Awesome icons (CDN)

There is no backend and no build step.

## How to Run

Open `index.html` in a browser.

An internet connection is needed to load Tailwind CSS, Google Fonts and Font Awesome.

## The Assignment Problem

The input is a **cost matrix**.

- Each row is a worker (agent).
- Each column is a job (task).
- Each cell is the cost of giving that job to that worker.

Example:

```text
           Job 1  Job 2  Job 3  Job 4
Worker 1     9      2      7      8
Worker 2     6      4      3      7
Worker 3     5      8      1      8
Worker 4     7      6      9      4
```

Each worker must get exactly one job.

Each job must go to exactly one worker.

The goal is to find the matching with the **minimum total cost**.

Trying every possible matching is slow. For `n` workers there are `n!` matchings. For 10 workers that is more than 3.6 million.

The Hungarian algorithm finds the optimal answer in `O(n³)` time.

## How It Works

```text
Cost matrix
    ↓
Pad to a square matrix (if needed)
    ↓
Row reduction
    ↓
Column reduction
    ↓
Cover all zeros with the minimum number of lines
    ↓
Lines = n? ──No──→ Adjust the matrix ──┐
    ↓ Yes                               │
Pick one zero per row and column  ←─────┘ (repeat cover step)
    ↓
Optimal assignment
```

The key idea:

> Subtracting a number from a whole row or a whole column does not change which assignment is optimal.

Every assignment uses exactly one cell from each row and each column. So it loses the same amount. The algorithm keeps subtracting until the optimal assignment is made of zeros.

### Step 0: Pad the Matrix

The classic algorithm needs a **square** matrix.

If there are more jobs than workers, the app adds **dummy rows**.

If there are more workers than jobs, the app adds **dummy columns**.

Dummy cells have a cost of `0`.

```text
3 workers × 5 jobs
    ↓
Add 2 dummy workers
    ↓
5 × 5 matrix
```

A job assigned to a dummy worker is simply left unassigned.

### Step 1: Row Reduction

For each row:

1. Find the smallest value in the row.
2. Subtract it from every cell in the row.

Every row now has at least one `0`.

```text
9  2  7  8   (min 2)  →   7  0  5  6
6  4  3  7   (min 3)  →   3  1  0  4
5  8  1  8   (min 1)  →   4  7  0  7
7  6  9  4   (min 4)  →   3  2  5  0
```

### Step 2: Column Reduction

For each column:

1. Find the smallest value in the column.
2. Subtract it from every cell in the column.

Every column now has at least one `0`.

```text
7  0  5  6        4  0  5  6
3  1  0  4   →    0  1  0  4
4  7  0  7        1  7  0  7
3  2  5  0        0  2  5  0
↑
min 3
```

### Step 3: Cover the Zeros

Draw horizontal and vertical lines so that every `0` is covered.

Use the **smallest possible number of lines**.

By **Kőnig's theorem**, the minimum number of lines equals the maximum number of zeros that can be chosen with no two in the same row or column.

```text
If lines = n  → an optimal assignment exists among the zeros
If lines < n  → go to Step 4
```

### Step 4: Adjust the Matrix

When there are not enough lines:

1. Find the smallest value `k` that is **not covered** by any line.
2. Subtract `k` from every **uncovered** cell.
3. Add `k` to every cell covered by **two lines** (an intersection).
4. Leave cells covered by one line unchanged.

This creates at least one new zero. Go back to Step 3.

### Step 5: Extract the Assignment

Choose `n` zeros so that each row and each column has exactly one chosen zero.

These positions are the optimal assignment.

Add up the costs from the **original** matrix to get the total cost.

## Worked Example

Using the matrix above, after Steps 1 and 2:

```text
           Job 1  Job 2  Job 3  Job 4
Worker 1     4     [0]     5      6
Worker 2    [0]     1      0      4
Worker 3     1      7     [0]     7
Worker 4     0      2      5     [0]
```

Four independent zeros exist, so the assignment is optimal:

```text
Worker 1 → Job 2   cost 2
Worker 2 → Job 1   cost 6
Worker 3 → Job 3   cost 1
Worker 4 → Job 4   cost 4
                   ──────
Total              13
```

## Presets

The app includes these example matrices:

| Preset             | Size  | Optimal cost |
| ------------------ | ----- | ------------ |
| Classic 3×3        | 3 × 3 | 100          |
| Workers 4×4        | 4 × 4 | 13           |
| Rectangular (3×5)  | 3 × 5 | 31           |
| Random Matrix      | any   | varies       |

Optimal costs were checked by brute force over all possible assignments.

## UI Features

The demo provides:

- Editable cost matrix (click a cell to change its value)
- Matrix sizes from 2 × 2 up to 7 × 7, including rectangular sizes
- Random fill with values from 10 to 99
- **Next Step** to move through the algorithm one phase at a time
- **Auto Play** with adjustable speed (0.2s to 2.0s per step)
- **Solve** to run straight to the result
- Phase tracker and step log
- Colour highlighting for zeros, covered lines, dummy cells and the final matching
- Guide pop-up with real-world use cases
- Responsive layout for phones and desktops

### Colour Legend

```text
Green  → Optimal match
Amber  → Zero
Indigo → Active cell
Red    → Covered by a line
Grey   → Dummy (padding) cell
```

## Real-World Uses

- **Task assignment:** assign employees to projects at minimum cost or time.
- **Ride-sharing and delivery:** match drivers to pickup requests to minimise waiting time.
- **Robot fleets:** assign robots to tasks or charging stations.
- **Object tracking:** match detections in one video frame to the next frame.
- **Cloud computing:** place services on machines to minimise latency.

In production code, use a library instead of writing the algorithm yourself:

- Python: `scipy.optimize.linear_sum_assignment`
- Google OR-Tools: `LinearSumAssignment`

## Important Limitation

This project is an educational demo.

The current implementation **simplifies Steps 3 and 5**:

- **Step 3** covers every column that contains a zero. It does not find the true minimum number of lines. After column reduction every column has a zero, so the algorithm always goes straight to the result and Step 4 never runs.
- **Step 5** picks assignments greedily. For square matrices it takes the first free column in each row, even if that cell is not a zero.

Because of this, the app can show a **non-optimal** total.

For example, the Workers 4×4 preset shows a total of `18` (the diagonal), but the correct minimum is `13`.

A correct version needs:

- A real minimum line cover (for example, using a maximum matching on the zeros)
- An assignment step that only picks zeros, using augmenting paths

## Goal of the Project

The main goal is to show **step by step** how the Hungarian algorithm turns a cost matrix into an optimal assignment.

It helps explain:

- The assignment problem
- Cost matrices
- Padding rectangular problems
- Row and column reduction
- Covering zeros and Kőnig's theorem
- Matrix adjustment
- Optimal bipartite matching
