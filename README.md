# LP-Based Seam Carving

A from-scratch implementation of **seam carving** (content-aware image
resizing) that formulates each seam-removal step as a **linear program**
and solves it with [Gurobi](https://www.gurobi.com/), rather than the
classic dynamic-programming approach.

Originally built as the final project for *IE 411: Optimization of
Large-Scale Systems*.

<p align="center">
  <img src="tower.png" width="30%" alt="Original image">
  &nbsp;&nbsp;&nbsp;
  <img src="final_image.png" width="25%" alt="Resized image">
</p>


<p align="center"><em>Original (left) vs. after removing 25 seams (right)</em></p>

## How it works

An image is modeled as a directed acyclic graph:

- Every pixel `(r, c)` is a node.
- Each pixel has an edge to its three "downward" neighbors —
  `(r+1, c-1)`, `(r+1, c)`, `(r+1, c+1)` — so a seam can bend slightly
  as it descends the image.
- A single dummy sink node `k` is reachable from every pixel in the last
  row.
- The cost of an edge is the absolute difference in grayscale intensity
  between the two pixels it connects, so cheap paths correspond to
  visually uniform (low-energy) regions.

Finding the lowest-energy vertical seam from a fixed starting pixel to
the sink is then a **shortest-path problem**, solved as a min-cost flow
LP of value 1:

```
Primal                              Dual
  min   qᵀx                          max   dᵀy
  s.t.  Mx = d                       s.t.  Mᵀy ≤ q
        x ≥ 0                              y free
```

where `M` is the (oriented) node-edge incidence matrix, `q` is the vector
of edge costs, and `d` is `-1` at the source, `+1` at the sink, and `0`
elsewhere. The full derivation — decision variables, parameters, and the
complementary-slackness conditions used as a correctness check — is
written out in the notebook itself.

Each iteration:
1. Builds the graph for the current image size.
2. Solves the primal LP to find the optimal seam.
3. Removes the pixels on that seam, shrinking the image by one column.
4. Solves the dual LP and checks complementary slackness as a
   correctness sanity check.

Repeating this `N` times removes `N` of the least visually important
columns from the image.

## Repository contents

| File | Description |
|---|---|
| `seam_carving.ipynb` | The full project: problem formulation, code, and results. |
| `tower.png` | Sample input image. |
| `final_image.png` | Sample output after 25 seam removals. |

## Requirements

- Python 3.9+
- Jupyter (`pip install jupyterlab` or `notebook`)
- [Gurobi](https://www.gurobi.com/) with a valid license (free academic
  licenses are available)
- `pillow`, `numpy`, `networkx`

```bash
pip install pillow numpy networkx gurobipy jupyterlab
```

## Usage

1. Open the notebook: `jupyter lab seam_carving.ipynb`.
2. Edit the **Configuration** cell near the top — `IMAGE_PATH`,
   `SOURCE_COL`, `NUM_ITERATIONS`, `OUTPUT_PATH` — for your image/run.
3. Run all cells. The final resized image is saved to `OUTPUT_PATH` and
   displayed in the last cell.

## Notes & limitations

- Seams are removed starting from a fixed source column rather than
  searching over all possible starting columns for the globally
  cheapest seam — a straightforward extension.
- The graph is rebuilt from scratch every iteration, which is simple
  but not the fastest approach; a production version would update the
  graph incrementally instead.

## License

Released under the [MIT License](LICENSE).
