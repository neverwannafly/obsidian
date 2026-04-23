[[leetcode medium]] [[array]] [[greedy]]
## Intuition
- If there are even number of negative numbers, all can be made positive
- If there's odd number of negative numbers, we can shift the negative sign to the smallest number

## Approach
```cpp
// @leet start
class Solution {
public:
    long long maxMatrixSum(vector<vector<int>>& matrix) {
      long long maxSum = 0;
      int minNum  = 1e9 + 7;
      int negativeCnt = 0;
      for (int i=0; i<matrix.size(); i++) {
        for (int j=0; j<matrix[i].size(); j++) {
          if (matrix[i][j] < 0) {
            negativeCnt += 1;
          }
          minNum = std::min(minNum, std::abs(matrix[i][j]));
          maxSum += std::abs(matrix[i][j]);
        }
      }

      if (negativeCnt % 2 == 0) return maxSum;
      return maxSum - 2 * minNum;
    }
};
// @leet end

```