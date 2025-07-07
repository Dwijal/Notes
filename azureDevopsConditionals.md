# Azure DevOps YAML Pipelines: Conditionals Cheatsheet

This cheatsheet provides a quick reference for using conditions in your Azure DevOps YAML pipelines at the job level.

---

## Built-in Status Conditions

These conditions check the status of the jobs that the current job depends on.

| Condition           | Description                                                                                             |
| ------------------- | ------------------------------------------------------------------------------------------------------- |
| `succeeded()`       | **Default.** Runs only if all dependent jobs succeeded.                                                 |
| `succeededOrFailed()`| Runs if dependent jobs succeeded or failed, but **not** if the pipeline was canceled.                  |
| `failed()`          | Runs only if **any** dependent job failed.                                                              |
| `always()`          | **Always** runs, regardless of the dependency status or even if the pipeline was canceled. Useful for cleanup tasks. 🧹 |

---

## Custom Condition Functions

You can build custom logic using these functions inside a `condition` block.

| Function                 | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| `eq(value1, value2)`     | Checks if `value1` is **equal to** `value2`.                 |
| `ne(value1, value2)`     | Checks if `value1` is **not equal to** `value2`.             |
| `in(value, ...values)`   | Checks if `value` is **in** the provided list of values.     |
| `notin(value, ...values)`| Checks if `value` is **not in** the provided list of values. |
| `startsWith(string, sub)`| Checks if a `string` **starts with** the `sub`string.      |
| `endsWith(string, sub)`  | Checks if a `string` **ends with** the `sub`string.        |
| `contains(string, sub)`  | Checks if a `string` **contains** the `sub`string.         |
| `and(cond1, cond2)`      | Returns `true` only if **both** conditions are true.         |
| `or(cond1, cond2)`       | Returns `true` if **either** condition is true.              |

---

## Examples

Here’s how you can combine these in your pipeline.

| Scenario                                     | Example YAML Condition                                                                                             |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Run only on the `main` branch.               | `condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))`                              |
| Run based on a boolean parameter.            | `condition: and(succeeded(), eq('${{ parameters.runTests }}', 'true'))`                                           |
| Run only for CI builds.                      | `condition: and(succeeded(), in(variables['Build.Reason'], 'IndividualCI', 'BatchedCI'))`                           |
| Run for a pull request targeting `main`.     | `condition: and(succeeded(), eq(variables['Build.Reason'], 'PullRequest'), contains(variables['System.PullRequest.TargetBranch'], 'main'))` |
