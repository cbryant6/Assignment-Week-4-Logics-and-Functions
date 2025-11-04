# Password Update 

```python
import re
from typing import Tuple

SPECIAL = set("!@#$%^&*")

def has_run_of_k(s: str, k: int) -> bool:
    """
    True if any character repeats ≥ k times consecutively.
    """
    if k <= 0:
        return True
    if k == 1:
        return bool(s)
    # e.g., k=3 -> (.)\1{2,}
    return re.search(r"(.)\1{" + str(k - 1) + r",}", s) is not None


def password_score(pw: str) -> Tuple[str, int]:
    """
    Apply your previous rules and return (label, score).
    """
    score = 0
    n = len(pw)

    # length
    if n >= 12:
        score += 2
    elif 8 <= n <= 11:
        score += 1

    # variety
    has_lower = any(c.islower() for c in pw)
    has_upper = any(c.isupper() for c in pw)
    if has_lower and has_upper:
        score += 1
    if any(c.isdigit() for c in pw):
        score += 1
    if any(c in SPECIAL for c in pw):
        score += 1

    # penalties
    if any(c.isspace() for c in pw):
        score -= 2
    if has_run_of_k(pw, 3):  # was: re.search(r"(.)\1\1", pw)
        score -= 3

    # keep non-negative as in your previous version
    score = max(score, 0)

    # label
    if score < 3:
        label = "weak"
    elif score <= 4:
        label = "okay"
    else:
        label = "strong"

    return label, score


# quick check with prior tests
if __name__ == "__main__":
    tests = [
        "P@ssw0rd",
        "hunter2",
        "CorrectHorseBatteryStaple",
        "abcde Fgh",
        "A!!!A!!!A!!!",
    ]
    for t in tests:
        print(t, "->", password_score(t))

```

# Tic-Tac-Toe New Code Winner

```python

def find_winner(board: list[list[str]]) -> str:
    """
    Determine the winner of a square Tic-Tac-Toe board.

    Each cell is "X", "O", or "".
    Returns:
        "X"       -> if X wins
        "O"       -> if O wins
        "Draw"    -> if board full and no winner
        "Pending" -> if empty cells remain
    """
    n = len(board)

    # rows & cols
    for i in range(n):
        # row
        if board[i][0] != "" and all(board[i][j] == board[i][0] for j in range(n)):
            return board[i][0]
        # column
        if board[0][i] != "" and all(board[j][i] == board[0][i] for j in range(n)):
            return board[0][i]

    # diagonals
    if board[0][0] != "" and all(board[i][i] == board[0][0] for i in range(n)):
        return board[0][0]
    if board[0][n - 1] != "" and all(board[i][n - 1 - i] == board[0][n - 1] for i in range(n)):
        return board[0][n - 1]

    # empty cells?
    if any("" in row for row in board):
        return "Pending"

    return "Draw"


# sample board
board = [
    ["O", "X", "X"],
    ["X", "X", "O"],
    ["O", "X", "X"]
]

print("\n=== Tic-Tac-Toe Winner ===")
print("Winner:", find_winner(board))  # -> 'X'


```

# Tic Tac Toe New Board

```python

from typing import Tuple

def draw_board(board: list[list[str]]) -> None:
    """
    Print a square Tic-Tac-Toe board.
    Each cell contains 'X', 'O', or '' (printed as a blank).
    Example for 2x2:
    --- ---
    | X | O |
    --- ---
    |   | X |
    --- ---
    """
    n = len(board)
    if n == 0:
        print("(empty board)")
        return

    horiz = ("--- " * n).rstrip()
    for r in range(n):
        print(horiz)
        row_cells = []
        for c in range(n):
            cell = board[r][c] if board[r][c] != "" else " "
            row_cells.append(f" {cell} ")
        print("|" + "|".join(row_cells) + "|")
    print(horiz)


def _current_player_and_mark(board: list[list[str]]) -> Tuple[int, str]:
    """
    Player 1 uses 'X', Player 2 uses 'O'.
    If #X == #O -> Player 1's turn, else Player 2's.
    """
    xs = sum(cell == "X" for row in board for cell in row)
    os = sum(cell == "O" for row in board for cell in row)
    if xs == os:
        return 1, "X"
    else:
        return 2, "O"


def update_board(board: list[list[str]]) -> list[list[str]]:
    """
    Prompt the correct player for a move and update the board in-place.
    Rules:
      - Determine whose turn based on current counts.
      - If no empty cell remains, do nothing and return board.
      - Otherwise, repeatedly prompt for row/col (0-based) until a valid
        empty cell is provided; then place the mark and return the board.
    """
    n = len(board)
    if n == 0 or any(len(row) != n for row in board):
        raise ValueError("Board must be a non-empty square list of lists.")

    # Any empty space?
    if not any("" in row for row in board):
        print("No empty spaces left. Board is full.")
        return board

    player, mark = _current_player_and_mark(board)
    print(f"Player {player}'s turn ({mark}).")
    draw_board(board)

    while True:
        raw = input(f"Enter row and column (0..{n-1}) separated by space: ").strip()
        parts = raw.split()
        if len(parts) != 2:
            print("Please enter exactly two integers separated by a space.")
            continue

        try:
            r, c = int(parts[0]), int(parts[1])
        except ValueError:
            print("Indices must be integers. Try again.")
            continue

        if not (0 <= r < n and 0 <= c < n):
            print(f"Indices out of range. Use 0..{n-1} for both row and column.")
            continue

        if board[r][c] != "":
            print("That square is not empty. Choose another.")
            continue

        # Place the mark and show the updated board
        board[r][c] = mark
        print(f"Placed {mark} at ({r}, {c}).")
        draw_board(board)
        return board


# --- Example usage ---
if __name__ == "__main__":
    b = [
        ["X", "O", ""],
        ["",  "X", ""],
        ["",  "",  "O"],
    ]
    draw_board(b)
    update_board(b)
```

# Path Planning for Vacuum Robot

```python

from typing import List, Tuple, Optional

Coord = Tuple[int, int]

# ---------------- Core helpers ----------------

def in_bounds(r: int, c: int, rows: int, cols: int) -> bool:
    return 0 <= r < rows and 0 <= c < cols

def is_free(grid: List[List[str]], r: int, c: int) -> bool:
    return grid[r][c] != '#'

def dfs_path(grid: List[List[str]], cur: Coord, dest: Coord,
             visited: set[Coord], path: List[Coord]) -> bool:
    """
    Recursive DFS with backtracking.
    Appends cur to path. If dead-end, pops it before returning False.
    """
    r, c = cur
    path.append(cur)
    if cur == dest:
        return True

    visited.add(cur)

    # Explore in the required order: up, right, down, left
    for dr, dc in [(-1, 0), (0, 1), (1, 0), (0, -1)]:
        nr, nc = r + dr, c + dc
        nxt = (nr, nc)
        if (in_bounds(nr, nc, len(grid), len(grid[0]))
            and nxt not in visited
            and is_free(grid, nr, nc)):
            if dfs_path(grid, nxt, dest, visited, path):
                return True

    # Backtrack
    path.pop()
    return False

def find_path(grid: List[List[str]], start: Coord, dest: Coord) -> List[Coord]:
    """
    Returns a path (list of (r, c)) from start to dest, or [] if none exists.
    """
    if not grid or not grid[0]:
        return []
    visited: set[Coord] = set()
    path: List[Coord] = []
    if dfs_path(grid, start, dest, visited, path):
        return path
    return []

# --------------- Optional conveniences ---------------

def parse_grid(raw: List[str]) -> List[List[str]]:
    """Turn a list of strings into a list of list of chars."""
    return [list(row) for row in raw]

def find_marks(grid: List[List[str]]) -> Tuple[Optional[Coord], Optional[Coord]]:
    """Find 'S' (start) and 'D' (dest) marks in the grid, if present."""
    s = d = None
    for r, row in enumerate(grid):
        for c, ch in enumerate(row):
            if ch == 'S':
                s = (r, c)
            elif ch == 'D':
                d = (r, c)
    return s, d

def draw_path_on_grid(grid: List[List[str]], path: List[Coord]) -> None:
    """
    Print the grid with the path overlayed as '*'.
    Keeps 'S', 'D', and '#' as-is.
    """
    overlay = [row[:] for row in grid]
    for (r, c) in path:
        if overlay[r][c] not in ('S', 'D', '#'):
            overlay[r][c] = '*'
    for row in overlay:
        print(''.join(row))

# ----------------- Main demo -----------------

if __name__ == "__main__":
    raw = [
        "S..#....",
        ".##.#..#",
        ".#...#..",
        ".#.#.#..",
        ".#.#.#D#",
        ".#.....#",
        ".#######",
        "........",
    ]
    grid = parse_grid(raw)

    # Option A: auto-detect S/D
    s, d = find_marks(grid)
    if s is None or d is None:
        raise ValueError("Grid must contain 'S' (start) and 'D' (dest) marks.")

    # Option B (if you want to supply coordinates directly):
    # s, d = (0, 0), (4, 6)

    path = find_path(grid, s, d)
    if path:
        print("Path found (row, col):")
        print(path)
        print("\nGrid with path (*):")
        draw_path_on_grid(grid, path)
    else:
        print("No path exists.")

```