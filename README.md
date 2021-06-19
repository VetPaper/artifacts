# VET: Identifying and Avoiding UI Exploration Tarpits

*We're in the process of populating this repository. If you don't see something you're looking for, please check back later.*

## Extended explanation of how VET addresses exploration tarpits

- For *Exploration Space Partition* regions, VET prevents the action that is observed immediately before the region on the trace, by instructing OurInfra to disable the UI element on which the action is exerted during testing.
- For *Excessive Local Exploration* regions, apart from the action disablement, VET additionally finds the most frequently occurring screen in each region, and instructs OurInfra to restart the target app when the screen is observed during testing.
- Additionally, OurInfra automatically restarts the target app when no action is observed within 30 seconds to cover the cases where the tool stops working (such situations cannot be covered using techniques from the first two points).

## Accessing raw data

All our raw experimental data are available
[here](https://github.com/VetPaper/artifacts/releases/tag/fse2021).

### Explanation of data format

Each Bzip2 tarball represents data for a single trace, with filenames in the form of `{TOOL_NAME}-{APP_NAME}-{RUN_ID}.tar.bz2`. The `{RUN_ID}` is encoded in the following fashion:

- `1`, `2`, and `3`: the initial runs for each tool-app pair.
- `4`, `5`, and `6`: the comparison runs, using exactly the same settings as initial runs.
- `vXdY`: VET-enhanced runs by addressing top-`X` ranked regions on the respective tool-app pair. These regions are all from initial runs. `Y` is any of `1`, `2`, and `3`.

Within each trace the meanings of files are as follows:

- `./{TIMESTAMP}.json` represents the full UI hierarchy of each screen in the trace. Within each file:
  - Each JSON object is a UI element. Child elements are stored as JSON arrays in the `ch` field.
  - The UI element that the tool acts on is marked with `is_source: true`. Note that it is possible to act without involving any UI element (e.g., pressing Back on the device).
  - `act_id` denotes Activity ID.
  - `ua_type` denotes what kind of action triggers OurInfra to record this step. The values have the following meanings:
    * 0: short click
    * 1: long click
    * 2: touch
    * 3: context click
    * 7: menu click
    * 100: back
  - `en`: whether the UI element is enabled.
  - `vclk`, `vlclk`, `vcclk`: class names of short click handlers, long click handlers, and context click handlers, respectively. If one field is not present for some object, the corresponding UI element does not respond to the corresponding event.
  - `bound`: UI element boundary.
  - `class`: Class name of the type of the UI element.
  - `ucls`: Unified class name, must be a super class of `class`, either starts with `android.widget.` or is `android.webkit.WebView`. Might not exist in some cases.
  - `id`: Resolved resource ID in strings if the UI element has one.
  - `idn`: Resource ID in numbers, `-1` if the UI element does not have an associated resource ID.
  - `hash`: Internal unique representation of the object supporting the corresponding UI element in the app's internal UI data structures.
- `./crash.log`: Logcat entries that capture the app's crash logs.
- `./tool.log`: logs produced by the testing tool.
- `./minitrace/cov-{TIMESTAMP}.log`: Coverage information produced by [MiniTrace](http://gutianxiao.com/ape/install-mini-tracing) throughout the test run.

Additionally, `regions.tar.bz2` contains JSON files recording VET's identified exploration tarpit regions. Each file is in the form of `{TOOL_NAME}-{APP_NAME}-{RUN_ID}.json`. The format of these files is explained as follows:

- When `{RUN_ID}` is any of `1`, `2`, `3`, the JSON file contains a list of region start timestamps and end timestamps.
- When `{RUN_ID}` is `top3`, where `top3` indicates the aggregated results for all three traces from the initial runs of the tool-app pair, the file contains a list of list-of-actions to address the identified regions, sorted by the ranks of regions. Each list-of-action is a JSON array of strings, each being in the following form:
  - `{TOOL_NAME}-{APP_NAME}-{RUN_ID}/{TIMESTAMP}.json`: disable the UI element involved in this step in enhanced runs. If no UI element is involved in this step, restart the app when this screen is seen during enhanced runs.
  - `!{TOOL_NAME}-{APP_NAME}-{RUN_ID}/{TIMESTAMP}.json`: Simply restart the app when this screen is seen during enhanced runs.

## Contact

Please direct your questions to [Wenyu Wang](mailto:wenyu2@illinois.edu).
