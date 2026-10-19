

The required libraries are imported.

```python
import json
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

Data is read from the file. There is a sister script that produces mock data in a similar format, and that is being used at the moment.  
If data is provided by Jexin, from the real world, then `data = data['emotion_data']` is uncommented.

```python
f = open("data.json")
data = json.load(f)
# data = data['emotion_data']
```

Some constants are defined.
 - `MOVING_WINDOW` is half the width of the moving window.
The area of influence of each data-point extends this far in each direction.  
In other words, at any point in the final graph, only data-points within this distance are considered.
 - `NUM_POINTS` is the number of sample points for the final graph. Ideally, the final graph should be evaluated everywhere, but it is discretized for performance reasons.

```python
MOVING_WINDOW = 2500
NUM_POINTS = 200
```

Each person's data is in a separate list-of-dicts.
Each data-point, which is a dictionary, is converted to a list and put in a list with every other data-point, merging everyone's data.  
This data is then sorted and converted into a dataframe.

```python
datapoints = []
for person in data:
    for dp in person:
        datapoints.append([dp['timestamp'],dp['emotion']])

datapoints = sorted(datapoints,key=lambda x : x[0])
df = pd.DataFrame(datapoints,columns=['timestamp','emotion'])
```

The set of all emotions present in the dataframe is stored, and a dictionary is initialised to store the data that will be plotted.

```python
emo_set = sorted(set(df.emotion))
mw_stats = {emo:[] for emo in emo_set}
mw_stats['timestamp'] = []
```

Now, a function is defined that takes the timestamp of the data-point and the reference timestamp, and returns a weight based on the difference (normalised by `MOVING_WINDOW`).
The polynomial is defined between 0 and 1 such that:  
 - Full weight given to datapoints exactly at the reference timestamp: $y(0) = 1$  
- Negligible weight penalty to datapoints slightly offset from the reference timestamp: $y'(0) = 0$  
- No weight given to datapoints at the boundary of consideration: $y(1) = 0$  
- Negligible weight given to datapoints close to the boundary of consideration: $y'(1) = 0$
$$
\Rightarrow y = 2x^3 - 3x^2 + 1
$$


```python
def mw_filter(ts,ref):
    norm_diff = abs(ts-ref)/MOVING_WINDOW
    return 2*(norm_diff)**3 - 3*(norm_diff)**2 + 1
```

Since the graph is evaluated at discrete points, a loop is set up with start and stop timestamps.  
We have cheated a little bit here and taken one discretization point more than the constant set earlier. The maths just works out simpler this way.

```python
min_ts = df.timestamp.min()
max_ts = df.timestamp.max()
for i in range(NUM_POINTS+1):
    ref_ts = min_ts + i*(max_ts-min_ts)/NUM_POINTS
```

We first filter out the neighbourhood of influence, extract the timestamps, and convert each timestamp into its weight. The sum of these weights is the total weight, which is distributed among the emotions.
The total weight corresponding to each emotion is normalised by the total weight

```python
    mw_nbh = df[abs(df.timestamp - ref_ts) < MOVING_WINDOW]
    timestamps = mw_nbh.timestamp
    tri_fltr = timestamps.apply(lambda x : mw_filter(x,ref_ts))
    norm_factor = tri_fltr.sum()
    mw_stats['timestamp'].append(ref_ts)
    for emo in emo_set:
        mw_nbh_emo = mw_nbh[mw_nbh.emotion==emo]
        timestamps = mw_nbh_emo.timestamp
        tri_fltr = timestamps.apply(lambda x : mw_filter(x,ref_ts))
        mw_stats[emo].append(tri_fltr.sum()/norm_factor)
```

The stacked area plot is plotted, with x-axis being the list of reference timestamps, and the share of each emotion on the y-axis.

```python
plt.stackplot(mw_stats['timestamp'],np.array([mw_stats[emo] for emo in emo_set]),labels=emo_set)
plt.legend()
plt.show()
```

![Figure_1.png](Figure_1.png "A sample plot")