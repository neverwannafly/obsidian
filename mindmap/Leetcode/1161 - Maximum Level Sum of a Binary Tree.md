[[leetcode medium]] [[BFS]]
## Intuition
Very standard BFS problem, traverse the tree in level order and keep building sums of each levels as you go though

## Code
```c++
// @leet start
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
struct Node {
  int level;
  TreeNode *tn;
  Node(int l, TreeNode *tn) {
    this->level = l;
    this->tn = tn;
  }
};

class Solution {
public:
    int maxLevelSum(TreeNode* root) {
      std::vector<int> lvlSum;

      std::queue<Node> Q;
      Q.push(Node(1, root));

      while (!Q.empty()) {
        Node top = Q.front(); Q.pop();

        if (top.level > lvlSum.size()) {
          lvlSum.push_back(0);
        }

        lvlSum[top.level - 1] += top.tn->val;

        if (top.tn->right) Q.push(Node(top.level+1, top.tn->right));
        if (top.tn->left) Q.push(Node(top.level+1, top.tn->left));
      }

      int maxVal = -1 * 1e9;
      int minLvl = 1e9;
      for (int i=0; i<lvlSum.size(); i++) {
        if (lvlSum[i] > maxVal) {
          maxVal = lvlSum[i];
          minLvl = i + 1;
        } 
      }

      return minLvl;
    }
};
// @leet end

```