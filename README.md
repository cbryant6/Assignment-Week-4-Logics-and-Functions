#Password Update 

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


# quick check with your prior tests
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