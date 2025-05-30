During a breach into a Volnayan AI research node, Task Force Phoenix uncovered a dormant decision system—its logic locked behind a concealed execution path. Intelligence suggests it was used to authorize classified operations. The correct path must be uncovered before it's lost to blackout.


```
import joblib
import numpy as np
from sklearn.tree import export_text

# Load the model
model = joblib.load('tree_model.joblib')
tree = model.tree_
classes = model.classes_

# Find the leaf node for 'UNLOCK_FLAG_PATH'
flag_class_idx = np.where(classes == 'UNLOCK_FLAG_PATH')[0][0]
print(f"Index of 'UNLOCK_FLAG_PATH': {flag_class_idx}")

# Inspect all nodes
for node_idx in range(tree.node_count):
    if tree.children_left[node_idx] == -1 and tree.children_right[node_idx] == -1:  # Leaf node
        class_idx = np.argmax(tree.value[node_idx])
        if class_idx == flag_class_idx:
            print(f"Leaf node {node_idx} predicts 'UNLOCK_FLAG_PATH'")
            print(f"Node value: {tree.value[node_idx]}")

# Export the tree as text to understand the path
print("\nText representation of the tree (first 1000 lines):")
tree_text = export_text(model, feature_names=[f"feature_{i}" for i in range(model.n_features_in_)])
print("\n".join(tree_text.split("\n")[:1000]))  # Limit output for readability
```

We can load the pre-training decision tree model from `tree_model.joblib` using this we can use scikit-learns `DecisionTreeClassifier`

```
flag_class_idx = np.where(classes == 'UNLOCK_FLAG_PATH')[0][0]
print(f"Index of 'UNLOCK_FLAG_PATH': {flag_class_idx}")
```
- **What it does**: This finds the index of the UNLOCK_FLAG_PATH class in the classes_ array. For example, if model.classes_ = `['UNLOCK_FLAG_PATH', 'class_1', 'class_2', ...]`, then flag_class_idx is 0.
- **Output**: Index of 'UNLOCK_FLAG_PATH': 0
- **Why it works**: The np.where function searches for the index where classes == 'UNLOCK_FLAG_PATH'. Since there’s only one such class,`[0][0]` extracts the first (and only) index. This index is used later to identify leaf nodes predicting UNLOCK_FLAG_PATH.

```
for node_idx in range(tree.node_count):
    if tree.children_left[node_idx] == -1 and tree.children_right[node_idx] == -1:  # Leaf node
        class_idx = np.argmax(tree.value[node_idx])
        if class_idx == flag_class_idx:
            print(f"Leaf node {node_idx} predicts 'UNLOCK_FLAG_PATH'")
            print(f"Node value: {tree.value[node_idx]}")
```

**What it does**: This loops through all nodes in the decision tree (tree.node_count gives the total number of nodes) and identifies leaf nodes (nodes with no children, indicated by children_left == -1 and children_right == -1). For each leaf node, it:
- Retrieves the class predicted by the node using np.argmax(tree.value`[node_idx]`). The tree.value`[node_idx]` array contains the distribution of training samples at that leaf node for each class.
- Checks if the predicted class matches flag_class_idx (i.e., UNLOCK_FLAG_PATH).
- If it matches, prints the node index and the value array.

```
Index of 'UNLOCK_FLAG_PATH': 0
Leaf node 1 predicts 'UNLOCK_FLAG_PATH'
Node value: [[1. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0.
  0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0.
  1. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0.
  2. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0.
  3. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0.

Text representation of the tree (first 1000 lines):
|--- feature_1 <= -5000002.47
|   |--- class: UNLOCK_FLAG_PATH
|--- feature_1 >  -5000002.47

```

This shows the root node of the decision tree, which splits based on `feature_1 <= -5000002.47`. If this condition is true, the tree predicts UNLOCK_FLAG_PATH. If false, it continues to other branches (not shown in the truncated output)


So we can send that to the submission container for the flag.

```
echo "0.0,-5000003.0,0.0,0.0,0.0" | nc 94.237.50.221 37511
```

```
HTB{tr33_p4th_tr4c3d_and_tr1gg3r3d}
```
