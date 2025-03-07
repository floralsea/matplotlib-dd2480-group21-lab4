# Report for assignment 4

## Project

Name: matplotlib

URL: https://github.com/matplotlib/matplotlib

Matplotlib is a comprehensive library for creating static, animated, and interactive visualizations. Matplotlib can be used in Python scripts, Python/IPython shells, web application servers as well as various graphical user interface toolkits, which produces
high-quality figures for different types of input.

## Onboarding experience

For lab4, we chose a new project ([matplotlib](https://github.com/matplotlib/matplotlib)) instead of the previous one ([jsoniter](https://github.com/json-iterator/java))

Generally, the onboarding experience was much simpler than the last lab, which could of course have something to to with matplotlib being an ongoing project, whereas jsoniter was not. The [Setting up Matplotlib for development](https://matplotlib.org/devdocs/devel/development_setup.html#installing-for-devs) instruction was helpful, and shortly summarized the setup looked like this:

1. Fork the Matplotlib repository
2. Create a new venv environment
3. Install the Python dependencies defined in requirements.txt
4. Install Matplotlib in editable mode

The venv keeps the Matplotlib development environment seperate from the system-wide Python installation and doesn't affect other projects.

Overall, the repository seemed to be in good shape, with tests running fine, and no compilation errors or such like we encountered in the last lab.

## Effort spent

For each team member, how much time was spent in

|     | Xu Zuo    | Biming Wen    | Gustav Nordström | Gustav Wallin |
|:---:|:-----------:|:---------------:|:---:|:---:|
|1  |2  |2  |2  |2  |
|2  |1  |1  |1  |1  |
|3  |1  |0  |2  |5  |
|4  |20min  |15min  |0  |30min  |
|5  |10  |0  |6  |8  |
|6  |5  |0  |8  |3  |
|7  |5  |0  |1  |0  |
|8  |1  |0  |0  |1  |

1. plenary discussions/meetings;

2. discussions within parts of the group;

3. reading documentation;

4. configuration and setup;

5. analyzing code/output;

6. writing documentation;

7. writing code;

8. running code?

For setting up tools and libraries (step 4), enumerate all dependencies
you took care of and where you spent your time, if that time exceeds
30 minutes.

## Overview of issue(s) and work done.

Title: \[Bug\]: unaligned multiline text when using math mode

URL: https://github.com/matplotlib/matplotlib/issues/29527

Summary in one or two sentences

When rendering multiline text where one line includes math mode (e.g., `$...$`), while another does not, the lines are not perfectly aligned. Specifically, the line containing math mode text appears slightly misaligned compared to the non-math text line, leading to an unintended visual offset.

Scope (functionality and code affected).

**Functinality**: This issue affects the rendering of multiline text in Matplotlib, particularly when mixing regular text and math mode text. The misalignment causes inconsistent vertical positioning of the text lines.

**Code affected**: The issue is primarily related to the `matplotlib.text.Text` class (in `text.py`), particularly the `_get_layout()` method, which is responsible for computing the layout of multiline text. It also involves `MathTextParser.parse()` in `_mathtext.py`, where differences in baseline and height calculations between math mode and regular text lead to misalignment. Additionaly, it relates to how the `backend` works/is when matplotlib running on different platforms (e.g., `linux`, `macos` or `windows`). Specifically, `draw_mathtext()` and `draw_text()` (see in `backend_agg.py` as an example) directly affect it, and a tricky function is `_get_text_metrics_with_cache()` which is called in `_get_layout()` in `text.py`, since there are the **same characters** in the two lines in this issue.

**Work done**

- Introduced a new attribute/field (`self.exist_math`) for each object from `Text Class` in `text.py`. The initial value is `False`, when creating a new `text` object from `Text Class` with `math text`, then `self.exist_math = True`.

- Refactored `draw()` in `text.py`. We first check whether the input text contains `math text` or not, if so, `self.exist_math = True`. Then we use this field as a parameter when calling `draw_text()`.

- Refactored `draw_mathtext()` in `backend_agg.py` and `_get_layout()` in `text.py`. We added new conditional judgment, if there's math text, we treat the entire text as math text for further calculating.

- Our solution works for the aim [issue](https://github.com/matplotlib/matplotlib/issues/29527), though it's not a perfect/solid one.

## Requirements for the new feature or requirements affected by functionality being refactored

Optional (point 3): trace tests to requirements.

### Existing Tests

In the original Matplotlib repository, `backend_agg.py`, `text.py` and `_mathtext.py` are directly related to [`issue#29527`](https://github.com/matplotlib/matplotlib/issues/29527). Although when we ran `pytest lib/matplotlib/tests` and the corresponding tests (`test_agg.py`, `test_text.py` and `test_mathtext.py`) passed, we looked close to the test code and found that no test covered the issue, i.e., **mixed math and normal text alignment issues**. Thus, we introduced new tests here.

### Updated/New Tests

To validate our fix for issue **#29527**, we introduced a new automated test case in `test_mixedmathtext.py`. This test ensures that when a string contains both **math expressions and normal text across multiple lines**, the **vertical bars (`||`) remain visually aligned** after the fix.

#### Test Implementation:
1. Create a Matplotlib figure and add a text object containing `"$k$1||\n1||"`.
2. Trigger layout computation by calling `fig.canvas.draw()`.
3. Extract the rendered text dimensions:
4. Use `MathTextParser.parse()` to separately compute the **widths of each line** (`$k$1||` and `1||`).
5. Calculate `x_offset = abs(width_math - width_normal)` to measure the X-axis misalignment of the `||` characters.
6. Ensure `x_offset < 1px`, verifying that the vertical bars are correctly aligned after the fix.
Additional Considerations:

#### Test Parameterization:
- The test runs for both `usetex=False` (MathText) and `usetex=True` (LaTeX mode) to ensure correctness across different rendering backends.
- Automated Testing Integration:
This test is part of the Matplotlib automated test suite, ensuring that any future modifications to text rendering do not reintroduce this issue.

  run:
  ```shell
  pytest lib/matplotlib/tests/test_mixedmathtext.py 
  ```

### Trace to Requirements

- Requirement:

  When a multi-line text contains a math expression, the text should be rendered consistently to ensure aligned vertical elements (e.g., || should not be misaligned across lines).

- Test Validation:

  - Before the fix:
Inconsistent handling of mixed math and normal text caused || to be misaligned.
  - After the fix:
If one line contains math text, the entire block is processed as math text, ensuring alignment.
  - The test confirms this by checking that x_offset < 1px, meaning the visual alignment of || is corrected.

- Impact on Matplotlib Rendering:

  Ensures consistent text layout when mixing math and normal text across multiple lines. Prevents regressions in both MathText (usetex=False) and LaTeX (usetex=True) rendering modes.

## Code changes

### Patch

Optional (point 4): the patch is clean.

For a clean patch, please refer to [`Commit 49b7d8a`](https://github.com/floralsea/matplotlib-dd2480-group21-lab4/commit/49b7d8a758991904d77fec08ae3f85eb81532fb9.patch).

However, the above commit is for the optional point 4, so generally there're no code changes, the main change is "removing unused code and adding comments". The **code changes** are mainly in the following commits: the first try is [`Commit 91a389d`](https://github.com/floralsea/matplotlib-dd2480-group21-lab4/commit/91a389d9754209c6abf2ea96dee2297cbc227f5c) and a better solution is [`Commit ad1196e`](https://github.com/floralsea/matplotlib-dd2480-group21-lab4/commit/ad1196ef9255e908b3db30f05a19d856812317cc#diff-2889b798b7f9a072d882bef4f0fc033db63538a9317df59fa963e9a82401bf40)

Alternatively, run `git diff 03b74ea 49b7d8a lib/matplotlib/text.py lib/matplotlib/backends/backend_agg.py`.

Optional (point 5): considered for acceptance (passes all automated checks).

## Test results

For the original test results, you can refer to [`./result_images`](https://github.com/floralsea/matplotlib-dd2480-group21-lab4/tree/main/result_images). Since the test results were rendered into **images**, the test results 
were under [`./result_images`](https://github.com/floralsea/matplotlib-dd2480-group21-lab4/tree/main/result_images). Although there were some **fails** when we deployed it locally, these fails didn't affect our target issue. To be specific, [`test_agg`](https://github.com/floralsea/matplotlib-dd2480-group21-lab4/tree/main/result_images/test_agg) and [`test_text`](https://github.com/floralsea/matplotlib-dd2480-group21-lab4/tree/main/result_images/test_text) are directly affected by our refactoring.

After our refactoring, we were able to pass most of the original tests. Only two tests in `test_text.py` are still failing, related to a ***parse_math*** flag not working properly when placing text in a figure. In the failing test, text which should not be parsed as math still is when the flag is not set, which has a connection to our solution since a lot of regular text will be in math mode. This would have to be resolved properly in order for the bugfix to be accepted.

For the test results, you can refer to [`./resources`](https://github.com/floralsea/matplotlib-dd2480-group21-lab4/tree/main/resources), where there is both images showing the bug solved and test logs from the tests. There are logs for both the tests difectly impacted by the bugfix ([`test_text.log`](resources/test_text.log), [`test_agg.log`](resources/test_agg.log), [`test_mathtext`](resources/test_mathtext.log)) and for the whole program, before and after the bugfix ([`test_log_full_before_fix`](resources/test_log_full_before_fix.log), [`test_log_full_after_fix`](resources/test_log_full_after_fix.log))

## UML class diagram and its description

The following UML diagram illustrates the key changes introduced in our fix for issue **#29527**. The modifications primarily affect the **Text class** in `text.py`, the `draw_mathtext()` method in `backend_agg.py`, and the **text layout calculations** in `_get_layout()`.

![uml class diagram](resources/uml.svg)

#### Key updates reflected in the diagram:

- New attribute `exist_math`

  Introduced in the `Text class` (`text.py`).
Initially set to `False`, but updated to `True` if **math text** is detected.

- Refactored `draw()` method in `text.py`

  Now checks if the input text contains **math expressions** and updates `exist_math` accordingly.
Passes this information to `draw_text()` for further rendering.

- Changes in `backend_agg.py` (`draw_mathtext()`)

  A conditional statement was added to ensure that if any line contains **math text**, the **entire text block** is processed as math text.

- Modifications in `_get_layout()` (`text.py`)

  Adjusted how text layout is computed based on the `exist_math` flag.

**Note**: We use the online tool [PlantUML](https://www.planttext.com/) to produce/draw our class diagram.

### Key changes/classes affected

Optional (point 1): Architectural overview.

We wrote the archtectural overview before and after our fix in [dd2480_lab4_option_1.pdf](resources/dd2480_lab4_option_1.pdf), please refer to it for detailed information.

Optional (point 2): relation to design pattern(s).

You can find the sub-report in [./resources/option2.md](resources/option2.md)

## Overall Experience

The main take-aways from this project have to do with working on a codebase this large and navigating the difficulties that come with it.

Firstly, understanding how Matplotlib handles text rendering is a challenge in itself. We've learned that identifying parts of the program that may cause the error you're looking for, and finding potential solutions in an unfamiliar code base without breaking anything else, can be tough. We understood that we had to get a deep understanding of the problem before jumping into solutions. Knowing how different parts of the program interact with eachother is essential when assessing the issue.

Additionally, we've gained more experience working on open-source projects. With collaborators from all over the world, we've been able to see first hand the clear communication that's needed in documentation, pull requests, commit messages etc. This definitely contrasts with the software development environments we're mostly used to, where we typically work in small groups and communication is more direct and easier.

### Essence Standard

Throughout this course, we've continously used the Essence standard to evaluate or performance and growth as a team.

We started with outlining our responsibilities and setting up some ground rules. We got a grasp if things quite quickly, and entered the Formed and then the Collaborating stage quite quickly.

Generally, we've spent a lot of time in the Collaborating stage, having to work on improving communication and addressing minor inefficiencies. Over the last weeks however, we've at times entered the Performing stage. Altough there is always room for improvement, we've become more effective and improved our ability to quickly adapt to the changing context.

### Software Engineering Practice

Optional (point 6): Our work in context of the SEMAT kernel

We will put our work in context in relation to [The Solution Area of Concern](https://www.omg.org/spec/Essence/1.2/PDF#page=50). This area covers the practices needed to produce good quality software, and contains the following alphas:

* Requirements
* Software System

#### Requirements

Requirements are essential in defining what the solution must solve. In our case, the requirement was derived from the original issue on Github.

As for the [Checklist for Requirements](https://www.omg.org/spec/Essence/1.2/PDF#page=54), we've achieved the Acceptable state. The requirement is clear, it is easily testable and the benefit in solving it is easy to understand. In the next states Addressed and Fullfilled, the requirement should communicate a clear value for implementing it and it also demands that a solution is partly accepted, something which we've not achieved.

One thing we haven't done to a great extent, and that is covered in the alpha, is continously working on improving the requirements. As time went on, it could have been beneficial to add more detail to the requirement, or even sub-requirements, when the task became more clear. 

#### Software System

The software system refers to the actual implementation and it working as intended.

As for our system, we've been able to implement a solution that works. However, it is not without fault, as we render non-math text as math. Our solution also had an effect on other test cases.

As for the [Checklist for Software System](https://www.omg.org/spec/Essence/1.2/PDF#page=58), we believe we've achieved parts of the Usable state. We've solved the problem, but not in a way that would be accepted. To achieve greater states, the solution would have to be fully operational, without the less ideal parts of our implementation.

This process shows us that even a seemingly small fix can be complicated to implement and have widespread consequences. It also highlights the need to iteratively refine the solution for it to meet the requirements.

