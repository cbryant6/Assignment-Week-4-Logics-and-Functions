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

# Tic-Tac-Toe New Code

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


