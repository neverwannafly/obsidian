[[dynamic programming]] [[leetcode hard]]

## Intuition
- Initially I did the mistake of assuming that all states are independent and came up with the following recurrence relation where i had ironed out 12 patterns and i was effectively doing a top up approach of enumerating next branches from current pattern 
  ```c++
  dp[i][k] = dp[i][k] + dp[i-1][j]
  ```
- But the idea was pretty flawed as the branching behaves different when row has 2/3 colors. At some row I,J if the count is 20, i have no idea what the structure of previous pattern was, if i forward 20 i'm counting the final result incorrectly
- the correct way to represent the dp state is `dp[i][col1][col2][col3]`, this essentially reduces the problem to a memoisation problem
## Solution
- the current solution iterates over all possible states of our recurrence relation and returns the answer
- We dont need the full memory, instead we can just keep track of the last two rows
- One more optimization that we can do, instead of enumerating over all possible states, we really need to take care of two combinations - when all 3 colors are different and when two colors are the same. Irrespective of the color, they will behave similarly
```c++
// @leet start

const int MOD = (int)1e9 + 7;
const int rows = 3;

std::vector<std::vector<int>> nextRows(int a, int b, int c) {
  std::vector<std::vector<int>> res;
 
  for (int i=0; i<rows; i++) {
    for (int j=0; j<rows; j++) {
      for (int k=0; k<rows; k++) {
        if (i != j && j != k && i != a && j != b && k != c) {
          res.push_back({i, j, k});
        }
      }
    }
  }
  return res;
}

class Solution {
public:
    int numOfWays(int n) {
      int dp[n][3][3][3];

      for (int i=0; i<n; i++) {
        for (int a=0; a<rows; a++) {
          for (int b=0; b<rows; b++) {
            for (int c=0; c<rows; c++) {
              if (i == 0) {
                dp[i][a][b][c] = (a != b) && (b != c);
              } else {
                dp[i][a][b][c] = 0;
              }
            }
          }
        }
      }



      for (int i=0; i<n-1; i++) {
        for (int a=0; a<rows; a++) {
          for (int b=0; b<rows; b++) {
            for (int c=0; c<rows; c++) {
              if (a == b || b == c) continue;
              std::vector<std::vector<int>> nr = nextRows(a, b, c);             
              for (int q=0; q<nr.size(); q++) {
                dp[i+1][nr[q][0]][nr[q][1]][nr[q][2]] = (dp[i+1][nr[q][0]][nr[q][1]][nr[q][2]] + dp[i][a][b][c]) % MOD;

              }
            }
          }
        }
      }
      
      long ans = 0;

      for (int a=0; a<rows; a++) {
        for (int b=0; b<rows; b++) {
          for (int c=0; c<rows; c++) {
            ans = (ans + dp[n-1][a][b][c]) % MOD;
          }
        }
      }

      return ans % MOD;
    }
};
// @leet end

```
