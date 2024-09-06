---
title: "Full Of Stars (HTB writeup)"
tags: [AI, HTB, Security, writeups]
date: 2024-09-06
---

# Full Of Stars (HTB writeup)







By Mark Dixon
## Background

**SPOILER ALERT:** This writeup contains spoilers.

---

This is a writeup for the Hack The Box AI challenge [Full of Stars](https://app.hackthebox.com/challenges/Full%2520of%2520Stars). My approach feels like a bit of a hack but it finds a solution in about 15 seconds (on a M1 CPU) which is the best I have been able to achieve.

If you have a more elegant or faster solution I'd love to hear about it!
## Intro

The challenge description is:

```nothing
While density scanning the galaxy we've found something extraordinary! It seems that the stars in our local group are communicating a message: The stars, spread over 256 clusters, were seen to blink in a specific order. However, in our charts we have only one star per cluster to uniquely identify the cluster, which leaves us with >99% stars uncategorized. Could you assist our researchers by identifying what stars belong to which cluster and deciphering the message left for us by mysterious extraterrestrial intelligence?
```

And the zip file contains two NPY files and a notebook with code to load the data and create the final jpg file. The loading code looks like this:

```python
import matplotlib.pyplot as plt
import numpy as np

data = np.loadtxt("data.npy.gz")
core_data = np.loadtxt("known_samples.npy.gz")
all_data = np.vstack((core_data, data))
del data

class_count = len(core_data)
out_labels = np.full((all_data.shape[0],), -1)
```

And the code to output flag.jpg is:

```python
# Assert that out_labels are within the correct range
assert(np.all(np.logical_and(0 <= out_labels, out_labels < 256)))

# Create the JPEG file
file_data = bytearray(out_labels[256:].shape[0])
for ix, val in enumerate(out_labels[256:]):
    file_data[ix] = val
file_data = bytes(file_data)
with open("flag.jpg", "wb") as outfile:
    outfile.write(file_data)
```

Just running this will give an raise an assertion exception and our job is to cluster the set of stars correctly.
## Visualising the data

Step one is to visualise the data to see what we are dealing with. I used the following Plotly code to give an interactive plot of all of the known points (in red and labeled) and the unknown points. 

```python
import numpy as np
import plotly.graph_objs as go
import plotly.io as pio
import plotly.colors as pc

valid_indices = np.where(np.where(out_labels == -1)[0])[0] # filter out the unabelled points
valid_labels = out_labels[valid_indices]
valid_stars = all_data[valid_indices]

# Separate core_data points from the valid stars
core_data_indices = np.arange(class_count)
core_data_stars = core_data  # The original labelled stars

# Choose a qualitative colorscale for distinct colors
color_scale = pc.qualitative.Plotly  # You can also try 'D3', 'Set1', etc.

# Create the 3D scatter plot for all stars
trace = go.Scatter3d(
    x=valid_stars[:, 0], y=valid_stars[:, 1], z=valid_stars[:, 2],
    mode='markers',
    marker=dict(
        size=1, color=valid_labels, colorscale=color_scale, opacity=0.8,
        colorbar=dict(title='Cluster Label')
    ),
    text=[f"Index: {i}<br>Label: {label}<br>X: {x}<br>Y: {y}<br>Z: {z}" 
          for i, (x, y, z), label in zip(valid_indices, valid_stars, valid_labels)],
    hoverinfo='text'  # Display the custom text on hover
)

# Create another scatter plot for the core_data stars with text annotations
core_trace = go.Scatter3d(
    x=core_data_stars[:, 0], y=core_data_stars[:, 1], z=core_data_stars[:, 2],
    mode='markers+text',
    marker=dict(size=2, color='red', opacity=0.9),
    text=[str(i) for i in core_data_indices],  # Display index as text
    textposition='top center'
)

# Set up the layout with increased height
layout = go.Layout(
    scene=dict(
        xaxis=dict(title='X Coordinate'),
        yaxis=dict(title='Y Coordinate'),
        zaxis=dict(title='Z Coordinate')
    ),
    width=900, height=900
)

fig = go.Figure(data=[trace, core_trace], layout=layout)
pio.show(fig)
```

This gives us the following:



![image](/20240906_154400_CleanShot_20240906_at_15.43.45.png)

Things that are apparent from an initial inspection:

- We are given one known member of each cluster (red dots)
- These known members are not the centroid of the cluster but randomly chosen
  members
- Most clusters are concave or have some property that means a simple
  quantization approach (cluster by radius from centroid) isn't going to work
## Normalising the data

The scale of the three axis is not normalised - the X axis ranges from -300 to +300, Y from -1500 to +1500 and Z from -100 to +100. So my first step was to normalise the points so that distances are the same regardless of the orientation of the cluster in space.

```python
# Normalising the data for X, Y, Z
x_min, x_max = np.min(all_data[:, 0]), np.max(all_data[:, 0])
y_min, y_max = np.min(all_data[:, 1]), np.max(all_data[:, 1])
z_min, z_max = np.min(all_data[:, 2]), np.max(all_data[:, 2])

# Apply normalisation to each axis across the entire dataset
all_data[:, 0] = (all_data[:, 0] - x_min) / (x_max - x_min)
all_data[:, 1] = (all_data[:, 1] - y_min) / (y_max - y_min)
all_data[:, 2] = (all_data[:, 2] - z_min) / (z_max - z_min)
core_data[:, 0] = (core_data[:, 0] - x_min) / (x_max - x_min)
core_data[:, 1] = (core_data[:, 1] - y_min) / (y_max - y_min)
core_data[:, 2] = (core_data[:, 2] - z_min) / (z_max - z_min)
```

Now each axis is from -1 to +1.
## Iterative search

Now we want to apply labels to all of the unknown stars based on their proximity to known stars.

Since the clusters are irregular an the initial known points are not centroid, I decided on an iterative search for nearest neighbours, labelling each neighbour with the known index and slowly expanding the maximum search distance for each iteration:

```python
from scipy.spatial import KDTree
import numpy as np

# Build KDTree using normalised data
kdtree = KDTree(all_data)

# BFS function to assign clusters using the full set of points with the same label at once
def assign_cluster(start_indices, cluster_label, out_labels, kdtree, max_distance):
    current_indices = np.array(start_indices)

    while True:
        # Find all stars within the max distance using KDTree for the current set of points
        nearby_indices = kdtree.query_ball_point(all_data[current_indices], max_distance, p=1, eps=0, workers=-1)

        # Flatten and deduplicate the list of nearby stars
        nearby_indices = set(np.hstack(nearby_indices))

        # Find the unprocessed points (those with -1 in out_labels)
        unprocessed_indices = [idx for idx in nearby_indices if out_labels[idx] == -1]

        # If no new unprocessed points are found, break the loop
        if not unprocessed_indices:
            break

        # Assign the cluster label to the unprocessed points
        #print(unprocessed_indices)
        for idx in unprocessed_indices:
            out_labels[idx] = cluster_label

        # Update current_indices for the next iteration
        current_indices = np.array(unprocessed_indices)

# Initialize out_labels with -1 to indicate unprocessed stars
out_labels = np.full((all_data.shape[0],), -1)

# Slowly grow the distance and assign clusters
for counter in range(10):  # grow the distance gradually
    print(f"Iteration: {counter}")
    for idx in range(class_count):
        out_labels[idx] = idx
        matching_indices = np.where(out_labels == idx)[0]
        assign_cluster(matching_indices, idx, out_labels, kdtree, 0.002 * counter)

```

Changing the plotly code to render the points whose index is not -1 (np.where(np.where(out<sub>labels</sub> != -1)[0])[0]) and re-running gives us nicely clustered stars:



![image](/20240906_155624_CleanShot_20240906_at_15.56.03.png)
## Flag

And now running the flag output code gives us a nice jpg:



![image](/20240906_155748_CleanShot_20240906_at_15.57.35.png)
