---
layout: post
title:  "Tackling the ARC Prize"
author: BR Patel, Minh Le
date:   2025-02-17 23:40:17 -0700
categories: research
---

# Tackling ARC
Over the last few months, we tackled [ARC-AGI](https://arcprize.org/arc) - a benchmark of visual tasks where given a small number of input-output examples, a model must predict the output for a test input. Most exciting to us was that the benchmark had resisted breakthroughs from large frontier models up through the middle of 2024 - the performance gaps between humans and AI remained large. To us, this indicated that conventional scaling methods at the time seemed unfruitful. Given our small team size and limited compute resource, we hoped that doing well in this benchmark would lead to innovation and insights into AI in compute constrained environments. We also were excited by the progress made by other small scale teams and the open community around it.  
# Our Approach
We wanted to explore 2 capabilities in tackling ARC: 1. Program synthesis 2. Iterative improvement on a solution given external observations and tool usage. We opted for induction style models (generating programs) rather than transduction style models (generating outputs directly) so that we could examine what we believe are hallmarks of intelligence. Firstly, we believe that intelligence can compose previously learned skills to solve new, potentially more complex tasks. Code is an especially good medium in which to observe the creation of abstractions or compositions. Secondly, we believe intelligence reflects on feedback received from the environment to iterate on a solution. We are drawn to program synthesis because programs provide natural feedback through execution and we can observe the evolution of candidate programs the model produces. 

# Existing Work and Inspiration
Our approach was largely inspired by many existing bodies of work including leveraging code execution for feedback, synthetic datasets, and leveraging inference time compute. 

Leveraging code execution has been shown to boost performance in reasoning tasks. For example,  [PAL](https://arxiv.org/abs/2211.10435) demonstrated model tool usage through python code execution to solve math word problems. [BARC](https://github.com/xu3kev/BARC) demonstrated that solvers using an inductive approach (outputting code to represent functions) were more effective at solving certain ARC tasks than transductive solvers that generated outputs directly.

Synthetic datasets are another common approach to the ARC challenge to augment the small 400 task training dataset. [Re-ARC](https://github.com/michaelhodel/re-arc) developed a DSL to generate additional tasks that followed the rules and patterns of the original ARC training dataset. [BARC](https://github.com/xu3kev/BARC) synthesized novel ARC tasks through a workflow of prompting a stronger LLM like GPT-4 to “mix” tasks from a seed set to a novel description of a problem, synthesizing code + inputs from the description, and generating the task from the code. [ARChitect](https://github.com/da-fr/arc-prize-2024/blob/main/the_architects.pdf), winner of the [2024 ARC challenge](https://arxiv.org/abs/2412.04604), further augmented these datasets through a series of reversible transformations such as reflection, rotations, and color permutations. 

Finally, we saw the emergence of inference time compute to further boost performance. A whole family of Chain of Thought (https://arxiv.org/abs/2312.04474) methods showed prompting LLMs to reason with “thinking tokens” prior to answering a query leads to improvements in reasoning tasks like math word problems. [CRITIC] (https://arxiv.org/pdf/2305.11738) demonstrated tool usage like python code execution to generate feedback to self-critique answers and improved performance on math reasoning tasks. While we did not see too much self-improvement and reasoning style applied specifically to ARC, many of the top approaches leveraged inference time compute via test time training. ARChitect formulated a semi-supervised dataset through just the training examples of each task to further fine-tune before inference during evaluation. 

# First steps to Impasse 
Our exploration into building an induction type system had two main phases: first, a focus on program synthesis, and next on program improvement.

As a baseline, we first experimented with naive program synthesis approaches. We explored using Re-ARC DSL coupled with enumerative search but quickly realized the search space was intractably large. Inspired by BARC’s library and leaning onto LLMs with strong coding capabilities, we next explored simply prompting models such as Llama-3.1-8b to output programs to solve ARC tasks. The results were very poor, confirming the findings of others who had attempted the same approach for the ARC benchmark.

We next decided, inspired by STAR, that we would attempt to train a model on examples of sequential program improvement towards solving a given task. We hoped that this would induce the ability in the model to iteratively generate a program, with execution feedback along the way, to solve a task. The first key challenge for this approach, of course, was how to generate examples of sequential program improvement.

We attempted to elicit self-improvement through providing feedback from the environment such as program output/state. Naively, we tried variations of prompts that contained:
Original problem statement
Prior candidate programs generated by the model to solve the problem
The execution output of those candidate programs
A directive to improve on the candidates
We attempted this type of prompting with models such as Lllama-3.1-8b, BARC fine tuned model, gpt-4o, and gpt-o1-preview. Across the board, we observed practically no self-improvement (at best irrelevant changes to the program that did not change the output).

We provide here an example of the self-improvement prompt containing the problem statement, the first candidate program and its execution results, and the directive to improve.
```

Given input-output grid pairs as reference examples, carefully observe the patterns to predict the output grid for new test input. Each pair follows the same transformation rule. Grids are 2D arrays represented as strings, with cells (colors) separated by spaces and rows by newlines.
Here are the input and output grids for the reference examples:
Example 1:
Input: 
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Yellow Black
Red Black Black Black Black Green Black Blue Yellow Blue
Black Black Black Black Black Black Blue Black Black Black
Blue Yellow Black Black Black Black Black Black Black Blue
Black Black Black Black Red Black Black Black Red Black
Black Black Yellow Black Black Black Black Black Black Black
Blue Black Black Yellow Black Yellow Black Black Green Black
Black Black Black Black Black Black Black Black Black Black
Black Blue Red Blue Black Black Black Black Black Black

Output:
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Black Black
Blue Black Black Black Black Black Black Black Black Black
Blue Black Black Black Black Black Black Black Black Black
Blue Black Black Yellow Black Black Black Black Black Black
Blue Black Black Yellow Black Black Black Black Black Black
Blue Red Black Yellow Black Black Black Black Black Black
Blue Red Black Yellow Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black


Example 2:
Input: 
Black Black Black Black Black Yellow Black Green Green Black
Black Blue Green Black Black Black Green Black Black Black
Black Black Black Black Blue Black Black Blue Black Yellow
Green Black Black Black Red Black Black Black Red Black
Black Black Black Black Black Black Black Black Black Black
Black Black Blue Black Black Black Black Black Black Black
Black Black Red Black Black Black Black Red Black Black
Black Green Black Black Black Yellow Green Red Black Black
Black Black Black Blue Black Black Black Black Green Black
Black Black Yellow Black Black Yellow Black Blue Black Blue

Output:
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Green Black Black Black Black Black Black Black
Blue Black Green Black Black Black Black Black Black Black
Blue Black Green Black Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black


Example 3:
Input: 
Black Black Black Black Black Yellow Black Black Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Green Black Black Black Black Black Green Black
Black Blue Black Black Red Black Black Yellow Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Black Yellow Black Green Black Black Red Black
Black Red Black Black Black Black Black Black Black Black
Black Black Black Black Blue Black Black Black Yellow Black
Black Black Yellow Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Black Black

Output:
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Black Yellow Black Black Black Black Black Black
Black Black Black Yellow Black Black Black Black Black Black
Black Red Green Yellow Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black
Blue Red Green Yellow Black Black Black Black Black Black


Here is the input grid for the test example:
Input: 
Black Green Black Black Black Black Black Black Black Black
Black Red Green Black Black Red Yellow Black Black Black
Black Green Black Red Black Black Black Black Black Green
Yellow Black Black Blue Black Black Black Blue Black Black
Black Black Black Blue Black Black Black Red Black Black
Green Black Blue Black Green Black Black Black Black Black
Black Black Black Black Black Black Black Green Black Black
Black Black Black Black Black Black Black Black Black Black
Blue Black Black Black Black Black Blue Black Black Green
Black Black Black Black Red Yellow Black Red Yellow Red

You are to write a Python function `transform` that can convert any given input grid to its corresponding output grid based on the pattern observed in the reference examples.

Here are previous attempts of an implementation and their execution result.
Attempt 1

import numpy as np
from arc.core import Color

def transform(input_grid: np.ndarray) -> np.ndarray:
    # Create an output grid initialized to black
    output_grid = np.full(input_grid.shape, Color.BLACK)
    
    # Define the color mapping based on observed patterns
    # This mapping is derived from the reference examples
    color_mapping = {
        Color.GREEN: Color.GREEN,
        Color.RED: Color.RED,
        Color.YELLOW: Color.YELLOW,
        Color.BLUE: Color.BLUE,
        # Add more mappings as necessary based on patterns
    }
    
    # Iterate through the input grid and apply the transformation
    for i in range(input_grid.shape[0]):
        for j in range(input_grid.shape[1]):
            color = input_grid[i, j]
            if color in color_mapping:
                # Fill the output grid with the mapped color
                output_grid[i, j] = color_mapping[color]
            elif color == Color.BLACK:
                # Maintain black color in the output grid
                output_grid[i, j] = Color.BLACK
    
    # Apply additional transformations based on observed patterns
    # For example, we can propagate colors downwards or to the right
    for i in range(1, output_grid.shape[0]):
        for j in range(output_grid.shape[1]):
            if output_grid[i, j] == Color.BLACK and output_grid[i - 1, j] != Color.BLACK:
                output_grid[i, j] = output_grid[i - 1, j]
    
    return output_grid

Here are execution results of this program.
Wrong Execution of Example 1
Input: 
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Yellow Black
Red Black Black Black Black Green Black Blue Yellow Blue
Black Black Black Black Black Black Blue Black Black Black
Blue Yellow Black Black Black Black Black Black Black Blue
Black Black Black Black Red Black Black Black Red Black
Black Black Yellow Black Black Black Black Black Black Black
Blue Black Black Yellow Black Yellow Black Black Green Black
Black Black Black Black Black Black Black Black Black Black
Black Blue Red Blue Black Black Black Black Black Black

Output:
Black Black Black Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Yellow Black
Red Black Black Black Black Green Black Blue Yellow Blue
Red Black Black Black Black Green Blue Blue Yellow Blue
Blue Yellow Black Black Black Green Blue Blue Yellow Blue
Blue Yellow Black Black Red Green Blue Blue Red Blue
Blue Yellow Yellow Black Red Green Blue Blue Red Blue
Blue Yellow Yellow Yellow Red Yellow Blue Blue Green Blue
Blue Yellow Yellow Yellow Red Yellow Blue Blue Green Blue
Blue Blue Red Blue Red Yellow Blue Blue Green Blue

Wrong Execution of Example 2
Input: 
Black Black Black Black Black Yellow Black Green Green Black
Black Blue Green Black Black Black Green Black Black Black
Black Black Black Black Blue Black Black Blue Black Yellow
Green Black Black Black Red Black Black Black Red Black
Black Black Black Black Black Black Black Black Black Black
Black Black Blue Black Black Black Black Black Black Black
Black Black Red Black Black Black Black Red Black Black
Black Green Black Black Black Yellow Green Red Black Black
Black Black Black Blue Black Black Black Black Green Black
Black Black Yellow Black Black Yellow Black Blue Black Blue

Output:
Black Black Black Black Black Yellow Black Green Green Black
Black Blue Green Black Black Yellow Green Green Green Black
Black Blue Green Black Blue Yellow Green Blue Green Yellow
Green Blue Green Black Red Yellow Green Blue Red Yellow
Green Blue Green Black Red Yellow Green Blue Red Yellow
Green Blue Blue Black Red Yellow Green Blue Red Yellow
Green Blue Red Black Red Yellow Green Red Red Yellow
Green Green Red Black Red Yellow Green Red Red Yellow
Green Green Red Blue Red Yellow Green Red Green Yellow
Green Green Yellow Blue Red Yellow Green Blue Green Blue

Wrong Execution of Example 3
Input: 
Black Black Black Black Black Yellow Black Black Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Green Black Black Black Black Black Green Black
Black Blue Black Black Red Black Black Yellow Black Black
Black Black Black Black Black Black Black Black Black Black
Black Black Black Yellow Black Green Black Black Red Black
Black Red Black Black Black Black Black Black Black Black
Black Black Black Black Blue Black Black Black Yellow Black
Black Black Yellow Black Black Black Black Black Black Black
Black Black Black Black Black Black Black Black Black Black

Output:
Black Black Black Black Black Yellow Black Black Black Black
Black Black Black Black Black Yellow Black Black Black Black
Black Black Green Black Black Yellow Black Black Green Black
Black Blue Green Black Red Yellow Black Yellow Green Black
Black Blue Green Black Red Yellow Black Yellow Green Black
Black Blue Green Yellow Red Green Black Yellow Red Black
Black Red Green Yellow Red Green Black Yellow Red Black
Black Red Green Yellow Blue Green Black Yellow Yellow Black
Black Red Yellow Yellow Blue Green Black Yellow Yellow Black
Black Red Yellow Yellow Blue Green Black Yellow Yellow Black

Iterate on previous versions of Python function `transform` so that there will be more correct executions.
Pay attention to the wrong executions of Example cases to discover and correct flaws in previous versions of `transform`. Output only the python code.
Reminder: The function should accept a single argument of type Grid from arc.core.
Each element of Grid is an integer corresponding to the Color IntEnum from arc.core.
For ease of language processing, the input and output examples given above are written
using the color names, but the `transform` function should operate on type Grid.
It is okay for your response to include an explanation of the problem and
your approach, but your response must end with a code block that begins with:
```

We prompted the model with varying levels of difficulty and a number of programs drawn from the BARC set. After verifying that a given buggy program no longer solves the task, we fine-tuned Llama-3.1-8b to recover the original program given the task and the buggy program. We’ll refer to this fine-tuned model as the Llama-bug-undoer. Next, we used the same self-improvement prompt shown above, but this time we appended the original (correct program) to create a training example of taking an incorrect program and improving it so that it would solve the task in the context. We then evaluated Llama-bug-undoer on generating solutions for ARC problems outside of the training set, using the same self-improvement prompting set up. The results were still not promising - “fixes” ended up being either irrelevant or wrong. We hypothesize that the distribution of bugs (color changes, swapped indices, etc.) introduced when we created the training examples does not represent the types of improvement necessary to iterate on initial attempts at solving an ARC task. 


# Pivot to reproducing SOTA
As we were hitting a wall in pursuing program synthesis and self improvement, the ARC challenge competition came to a close, and we saw that the SOTA approaches were primarily transduction style models with no iteration on output. We decided that the best use of our next efforts would be to replicate SOTA systems to obtain a baseline understanding of how these systems work. Specifically, we reimplemented the ARChitect's approach - which included dataset augmentation, better task representation/constrained token space, a novel sampling and ranking strategy, and test time training. This was a rewarding experience. We achieved 49% as against their 51% for Llama-rearc using the top two solutions and a DFS cutoff of 10%, and more importantly we learned a good deal along the way.

# Next Steps
From our work in replicating ARChitect, we are less bullish on the necessity of an induction style approach - whether the model outputs a program to solve a task or internally is executing a program seems irrelevant in terms of achieving good performance.

However, we remain dissatisfied that the SOTA approaches pretty much still only zero shot solutions. Despite leveraging inference time compute through TTT, there was no self refinement/iteration within a single task which we believe would be necessary as the complexity of a task scales. 

Motivated by the impressive results of applying a purely reinforcement learning approach to induce reasoning chains in Deepseek R1 Zero, we plan to experiment with something similar for ARC. 

We believe most ARC problems are simpler in essence than most of the problems on benchmarks that small models distilled from Deepseek R1 perform well on. Therefore, we will be pursuing reinforcement learning to induce chain of code type program writing. We believe that given that the ARC problem space is relatively limited, this approach may yield great results even on a smaller model.
