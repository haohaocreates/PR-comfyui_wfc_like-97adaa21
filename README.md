# comfyui_wfc_like
An "opinionated" Wave Function Collapse implementation with a set of nodes for comfyui

<details>
<summary>Important Technicality </summary>
<br>
This implementation is not a pure-to-form implementation of the wave collapse algorithm.

The “wave” of possibilities is not kept and updated on the entire grid; instead, only the boundary of the collapsed nodes is evaluated, expanding the boundary at each iteration, and validating only the states of cells adjacent to the expanded ones. In this sense, it would be fair to name it something else, since instead of a wave of possibilities the algorithm only satisfies local constraints until reaching an impossible state, at which point it backtracks. Nevertheless, the wave-function-collapse captures and helps clarify the potential applications of this algorithm.

Additionally, in the spirit of being used as a visual tool, there is no way to specify global constraints, and all local constraints are inferred from the given samples.
Although this makes some sets of rules hard to specify, the envisioned application is not to arrive at a complete solution necessarily, but rather a partial one, which can be completed using diffusion. 

This implementation searches for possible states using a best-first search which also considers the node’s depth to make the search greedy towards already deep paths to speed up the generation towards a partially acceptable state, i.e. a state that hasn’t collapsed all the cells but should be somewhat complete, provided the constraints are not very intricate. 
</details>

## Custom Nodes

### Sample (WFC)

This node analyses an image made out of tiles and extracts: each tile type, their count, and all existing 3x3 tile arrangments (constraints).

This data can be sent via the *sample* output slot to generator nodes, to create "similar" images.

<details>
<summary>Required Inputs</summary>
<br>

`img_batch` : an image, or batch of images of the same tileset.
If given a batch as input, the node will only return a single output, where the tile counts and adjacencies in all the images in the batch are considered. If given a list, it will analyze each image, or batch, in the list, and the output will be a list.

`tile_width` & `tile_height`  : the width and height in pixels of a single tile.

`output_tiles` : if set to true, all the different tile types will be sent as an image batch via the *unique_tiles* output slot.

</details>

<details>
<summary>Outputs</summary>

<br>

`sample` : the data obtained from the input samples, used as input to generator nodes.

`unique_tiles` : image batch with the tile types.

</details>

### Generate (WFC)

Generates a state using the provided *sample* data and *starting_state* using wave-function-collapse.

<details>
<summary>Required Inputs</summary>

<br>

`sample` : the sample data with the possible tile neighborhoods (constraints), and frequencies.

`starting_state` : the starting state to generate, or complete.

`seed` : controls randomness, to ensure reproduceable outputs.

`max_freq_adjust` : the maximum possible weight for frequency adjustments.

If set to Zero ( 0 ), the tile frequency is the *sample* input is completely ignored. Depending on the constraints of a given *sample*, this might result in a seldom biased generation that tends to overplace a subset of tile types which better minimizes the entropy.

If set to One ( 1 ), the search will consider the difference in the tile frequencies of the generation versus the provided *sample*, and try to skew the generation to compensate for the differences. This can make the generation slower since selected tiles might be worse at minimizing entropy and skew the generation towards contradictions.

The frequency adjustment is not weighted equally at every step in the generation, and it can be negated if the generation has frequent and high-depth backtracks that suggest hard-to-satisfy constraints.
There is no way to adjust this behavior besides editing the source code.

`use_8_cardinals` : if set to true, all the 3x3 sections in the generation must correspond to an existing 3x3 tile neighborhood in the *sample* input. Otherwise, the diagonals are ignored.

`relax_validation` : if set to true, does not check if tiles adjacent to tiles considered for open positions retain a valid neighbor configuration. This allows for a potentially faster generation with fewer incomplete sections; however, it will likely generate some invalid 3x3 tile patches, that do not satisfy the given constraints.

`plateau_check_interval` : defines how many nodes to process, when exploring the possibilities tree, before checking the highest depth found so far. If two consecutive checks share the same highest depth then the search is stopped.

As an alternative to specifying a concrete interval, the following options can be used instead:

Zero ( 0 ): The search continues until either: a fully complete state is found; or all possible combinations have been explored.

Minus One ( -1 ): The interval is automatically set based on the size of the *initial_state*.
The goal is to strike a balance:
not too long (to avoid excessive runtime).
not too short (to prevent stopping prematurely).
Keep in mind that the effectiveness of this auto-setup may vary depending on your specific use case.

</details>

<details>
<summary>Optional Inputs</summary>

<br>

`custom_temperature_config` : temperature is a custom mechanic implemented to weight the random component, and frequency adjustment, of a node's cost. The temperature increases as backtracks increase in frequency and depth, lowering the influence of these components and favoring the most probable states to skew the generation away from contradictions.

Use the *Custom Temperature* custom node to constrain the temperature bounds and define the initial temperature. ( additional clarifications in the *Custom Temperature* node documentation )

`custom_node_value_config` : change the weight of the different components used to weight a node's value. Nodes with lower values are visited first.

By default, if not set by the user, the used weights are 1, 1, and 0.

Use the *Custom Node Value* custom node to create an alternative configuration. ( additional clarifications in the *Custom Node Value* node documentation )

</details>

<details>
<summary>Outputs</summary>

`state` : The generated state. 

To convert it to an image, use the *Decode (WFC)* node.

It can also be used as input to other nodes for additional processing.

</details>
