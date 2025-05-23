---
acl_categories:
- '@tdigest'
- '@read'
arguments:
- name: key
  type: key
- name: low_cut_quantile
  type: double
- name: high_cut_quantile
  type: double
categories:
- docs
- develop
- stack
- oss
- rs
- rc
- oss
- kubernetes
- clients
complexity: O(N) where N is the number of centroids
description: Returns an estimation of the mean value from the sketch, excluding observation
  values outside the low and high cutoff quantiles
group: tdigest
hidden: false
linkTitle: TDIGEST.TRIMMED_MEAN
module: Bloom
since: 2.4.0
stack_path: docs/data-types/probabilistic
summary: Returns an estimation of the mean value from the sketch, excluding observation
  values outside the low and high cutoff quantiles
syntax_fmt: TDIGEST.TRIMMED_MEAN key low_cut_quantile high_cut_quantile
syntax_str: low_cut_quantile high_cut_quantile
title: TDIGEST.TRIMMED_MEAN
---
Returns an estimation of the mean value from the sketch, excluding observation values outside the low and high cutoff quantiles.

## Required arguments

<details open><summary><code>key</code></summary> 
is key name for an existing t-digest sketch.
</details>

<details open><summary><code>low_cut_quantile</code></summary> 
  
Foating-point value in the range [0..1], should be lower than `high_cut_quantile`
  
When equal to 0: No low cut.
  
When higher than 0: Exclude observation values lower than this quantile.
</details>

<details open><summary><code>high_cut_quantile</code></summary> 
  
Floating-point value in the range [0..1], should be higher than `low_cut_quantile`  
  
When lower than 1: Exclude observation values higher than or equal to this quantile.

When equal to 1: No high cut.
</details>

## Return value

[Simple string reply]({{< relref "/develop/reference/protocol-spec#simple-strings" >}}) estimation of the mean value. 'nan' if the sketch is empty.

## Examples

{{< highlight bash >}}
redis> TDIGEST.CREATE t COMPRESSION 1000
OK
redis> TDIGEST.ADD t 1 2 3 4 5 6 7 8 9 10
OK
redis> TDIGEST.TRIMMED_MEAN t 0.1 0.6
"4"
redis> TDIGEST.TRIMMED_MEAN t 0.3 0.9
"6.5"
redis> TDIGEST.TRIMMED_MEAN t 0 1
"5.5"

{{< / highlight >}}
