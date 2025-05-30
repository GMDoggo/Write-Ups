# About
During an analysis of a compromised satellite uplink, a suspicious dataset was recovered. Intelligence indicates it may encode physical access credentials hidden within the spatial structure of Volnaya’s covert data infrastructure.


# Solve

The CSV contains 1822 entries and 4 columns.

We are given rows of spatial cords x,y,z and a the label column.

![](../Images/Pasted%20image%2020250523153141.png)

We can start mapping the cords in a 2d plane based on their label

```
import pandas as pd
import matplotlib.pyplot as plt

csv_path = 'uplink_spatial_auth.csv'

# Load data
df = pd.read_csv(csv_path)

# Filter by label
label_1 = df[df['label'] == 1]
label_2 = df[df['label'] == 2]

# Create figure with 2 subplots side by side
fig, axes = plt.subplots(1, 2, figsize=(12, 6))

# Plot label 1
axes[0].scatter(label_1['x'], label_1['y'], s=5, c='blue', alpha=0.7)
axes[0].set_title('2D Projection: Label == 1')
axes[0].set_xlabel('X')
axes[0].set_ylabel('Y')
axes[0].axis('equal')

# Plot label 2
axes[1].scatter(label_2['x'], label_2['y'], s=5, c='red', alpha=0.7)
axes[1].set_title('2D Projection: Label == 2')
axes[1].set_xlabel('X')
axes[1].set_ylabel('Y')
axes[1].axis('equal')

plt.tight_layout()
plt.show()

```

We get the following which appears to be a QR code

![](../Images/Pasted%20image%2020250523153540.png)

Looks like we are supposed to overlay them? Doing it for all labels gets us no where basically I think we could probably make `label==1 `readable

![](../Images/Pasted%20image%2020250523153712.png)

All labels stacked did not give anything

![](../Images/Pasted%20image%2020250523153920.png)

So we def have to make label 1 work.

Utilized some good LLM explanation for Claude to build an interactive app that takes all of the plot points and maps them out on the grid to start to visualize it better.

![](../Images/Pasted%20image%2020250523154653.png)

```
HTB{clu5t3r_k3y_l34k3d}
```
