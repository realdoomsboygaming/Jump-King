# Jump King Project Scratchpad

## Background and Motivation

The user wants to identify and fix bugs in the existing Jump King p5.js project codebase to improve stability and functionality.

## Key Challenges and Analysis

*   **Complexity:** Some files like `Player.js` and `LevelSetupFunction.js` are quite large, increasing the likelihood of hidden bugs.
*   **Dynamic Typing:** JavaScript's dynamic typing can lead to runtime errors that are harder to catch statically.
*   **State Management:** Games often involve complex state management (player position, physics, level state), which can be a source of bugs if not handled carefully.
*   **Lack of Automated Tests:** Based on the file listing, there don't appear to be dedicated test files. Relying solely on manual testing can miss edge cases.
*   **Potential Physics Issues:** Jump King relies heavily on precise physics. Bugs in collision detection, gravity, or jump mechanics could significantly impact gameplay.

## High-level Task Breakdown

1.  **Setup Static Analysis:** Introduce a linter (like ESLint) to automatically catch common JavaScript errors and style issues.
2.  **Analyze Large/Complex Files:** Manually review `Player.js` and `LevelSetupFunction.js` for potential logic errors or areas prone to bugs.
3.  **Test Core Mechanics:** Manually test critical game mechanics like jumping, collision detection, and level transitions. Document any observed issues.
4.  **Add Basic Tests (Optional but Recommended):** Introduce a simple testing framework (like Jest or P5.js's own testing capabilities if available) and write basic tests for key functions, especially in physics or state management.
5.  **Prioritize and Fix:** Based on the findings from static analysis, code review, and testing, prioritize the identified bugs and fix them systematically.

## Project Status Board

*   [X] Task 1: Setup Static Analysis (ESLint) - *Completed. 0 errors, 56 warnings.*
*   [ ] Task 2: Analyze `Player.js` - *In Progress*
*   [ ] Task 3: Analyze `LevelSetupFunction.js`
*   [ ] Task 4: Manual Test - Core Mechanics (Jump, Collision, Level Transition)
*   [ ] Task 5: Add Basic Tests (Optional)
*   [ ] Task 6: Prioritize and Fix Bugs

## Executor's Feedback or Assistance Requests

*   **Task 1 (ESLint Setup):**
    *   Completed. Final report (`eslint_report.txt`) shows 0 errors, 56 warnings.
*   **Task 2 (Analyze `Player.js`):**
    *   Started analysis. Focused on collision logic (`CheckCollisions`, `GetPriorityCollision`).
    *   Identified potential issues: recursive collision checks with hard limit, complex/fragile priority logic (with unused variables), intricate diagonal collision handling.
    *   Investigated unused variables in `GetPriorityCollision`:
        *   Removed unused `maxCorrection` variable.
        *   Noted that `playerCornerPos` calculation seems overly complex for current logic, which only prioritizes minimum correction distance. Suggests potentially incomplete or simplified prioritization logic.
    *   **Analyzed `CheckCollisions` Recursion:**
        *   Recursion occurs if >1 initial collision is detected (after resolving the highest priority one).
        *   Uses the *original* list of lines (`currentLines`) for the recursive call, not re-evaluating collisions based on the adjusted position.
        *   This could lead to incorrect physics or infinite loops (currently caught by `maxCollisionChecks`).
    *   **Proposed Refinement:** Replace recursion with a `while` loop. The loop continues as long as collisions are detected (up to `maxCollisionChecks`). Inside the loop:
        1.  Find all currently colliding lines based on the *current* position.
        2.  If none, break.
        3.  Get the highest priority collision.
        4.  Resolve it (adjust position/speed).
        5.  Handle potential landing state changes.
        6.  Loop continues to re-evaluate collisions with the new position.
    *   This approach ensures collision checks are always based on the latest position within the resolution process.
    *   **Next Step:** Implement the loop-based refinement for `CheckCollisions`.

## Lessons

*   ESLint v9+ defaults to `eslint.config.js` (flat config), not `.eslintrc.*`. Use `ESLINT_USE_FLAT_CONFIG=false` environment variable for legacy `.eslintrc.*` files.
*   Complex commands with environment variables might require shell-specific syntax (e.g., PowerShell `$env:VAR='value'; command` vs. `VAR=value && command`).
*   Large ESLint outputs might cause terminal rendering issues; redirecting to a file (`> eslint_report.txt`) is safer.
*   Start ESLint with only critical rules (`eslint:recommended`, `no-undef`) and disable stylistic rules initially to avoid excessive noise.
*   Remember to ignore vendor directories (like `libraries/`) in ESLint config.
*   Carefully check class definitions and scope when encountering `no-undef` for classes used across files (e.g., `DiagonalCollisionInfo` defined in `Line.js` but used in `Player.js`). Avoid unnecessary instantiation if objects already exist (e.g., modifying `line.diagonalCollisionInfo` instead of creating a new one).
*   Ensure helper classes have necessary methods (e.g., adding `reset()` to `DiagonalCollisionInfo`).
*   Globally declared variables/functions used across multiple files need to be added to ESLint `globals` (e.g., `levels`, `levelImages`, `setupLevels`). 