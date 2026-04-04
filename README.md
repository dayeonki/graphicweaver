<div align="center">

 # GraphicWeaver: Benchmarking Agentic Planning <br> for Graphic Design Generation

<p align="center">
<img width="917" height="510" alt="Screenshot 2026-04-03 at 2 43 16 PM" src="https://github.com/user-attachments/assets/dbb77210-7f9f-462e-adab-5c882d1a90c8" />
</p>


<a href=https://dayeonki.github.io/>Dayeon Ki</a><sup>1</sup>, <a href=https://tianyizhou.github.io/>Tianyi Zhou</a><sup>1</sup>, <a href=https://www.cs.umd.edu/~marine/>Marine Carpuat<a><sup>1</sup>, <a href=https://research.adobe.com/person/gangwu/>Gang Wu</a><sup>2</sup>, <a href=https://themadaiguy.github.io/>Puneet Mathur</a><sup>2</sup>, <a href=https://research.adobe.com/person/vishy-swaminathan/>Viswanathan Swaminathan</a><sup>2</sup>
<br>
<sup>1</sup>University of Maryland, <sup>2</sup>Adobe Research
<br>

This repository contains the code and dataset for our paper <br> **GraphicWeaver: Benchmarking Agentic Planning for Graphic Design Generation**.

<p>
  <a href="https://arxiv.org/abs/2504.11571" target="_blank" style="text-decoration:none">
    <img src="https://img.shields.io/badge/arXiv-Paper-b31b1b?style=flat&logo=arxiv" alt="arXiv">
  </a>
</p>

</div>

---

## 👾 TL;DR
While prior work on vision-language model agents has primarily focused on well-defined problems with explicit goals, the capabilities of agents in creative graphic design, where goals are inherently open-ended and subjective, remain largely underexplored.
We introduce **GraphicWeaver**, a planning benchmark for graphic design comprising 1,079 diverse user queries and associated images spanning four design categories.

## 📰 News
- **`2026-04-03`** We release our code and dataset!


## ✏️ Content
- [🗺️ Overview](#overview)
- [🚀 Quick Start](#quick_start)
  - [GraphicWeaver Benchmark](#graphicweaver-benchmark)
  - [Multi-Agent Framework](#multi-agent-framework)
  - [Evaluation](#evaluation)
- [🤲 Citation](#citation)
- [📧 Contact](#contact)

---

<a id="overview"></a>
## 🗺️ Overview

Recent advances of Vision-Language Models (VLMs) have expanded the potential for automating various human tasks. However, research on the planning capabilities of VLM agents for creative design tasks remains limited. **Design planning** involves translating a high-level user request into a structured workflow composed of executable sub-tasks that collectively produce the final design. This is inherently complex, posing multiple challenges in terms of handling user queries, planning workflows, and evaluating design outcomes:

1. Planning for a complex design often requires involvement of multiple expert agents.
2. Planning a design workflow is long-horizon, involving a large number of interdependent decisions across agents, with an expansive action space to explore.
3. Design planning must accommodate both explicit constraints from user queries (e.g., _the title text color must be white_) and implicit constraints inferred through commonsense reasoning (e.g., _the background should contrast with the color of text elements_) since user queries might be incomplete and lack explicit details necessary for planning.
4. Assessing design outcomes is inherently subjective, as the notion of _better_ design varies among individuals.

These challenges raise a key question: To what extent can VLM agents generate cohesive plans for creative design tasks when provided only with open-ended user queries?


### Results

<p align="center">
<img width="850" alt="Screenshot 2026-04-03 at 2 46 51 PM" src="https://github.com/user-attachments/assets/1a76bbca-3640-49cf-a448-f2186b9f6490" />
</p>


<a id="quick_start"></a>
## 🚀 Quick Start

### GraphicWeaver Benchmark

<p align="center">
<img width="850" alt="Screenshot 2026-04-03 at 2 48 40 PM" src="https://github.com/user-attachments/assets/dbad7a9b-c2d1-47f9-ac5f-abbc4e397f94" />
</p>
GraphicWeaver contains 1,079 pairs of diverse user queries and input images across four types of graphic design: book covers, business cards, postcards, and posters. The dataset is divided into training and test sets, with the training set containing 5 instances per design type with human-annotated reference plans (20 pairs in total) and the test set comprising 1,059 instances.

- **Training set:** `data/train.jsonl`
- **Test set:** `data/test.jsonl`
- **Human annotation interface:** `human_annotation/`



### Multi-Agent Framework
We propose a multi-VLM agent framework to evaluate the creative design planning abilities of VLM agents on GraphicWeaver. Specifically,

1. The supervisor agent recruits expert agents and forms an expert group.
2. Each expert agent generates individual workflow plans.
3. The supervisor agent integrates individual workflows into a cohesive plan.
4. The supervisor agent retrieves appropriate tools for each workflow step.
5. Each step in the planned workflow is executed to produce a final design outcome. For tool retrieval, we define a set of 46 tools executable within three web-based design tool environments.


#### (1) Planning phase

To run the planner code,
```bash
python -u planning/planner.py \
  --input_path $PATH_TO_INPUT_FILE \
  --output_path $PATH_TO_OUTPUT_FILE \
  --llm $LLM
```

Arguments for the planner code are as follows:
  - `--input_path`: Path to input data file.
  - `--output_path`: Save path of output file (after rewriting).
  - `--llm`: The name or path of a transformers-based pre-trained checkpoint. You can directly refer to the Huggingface model.

The output file will contain the intermediate results (recruitment status, individual workflows of expert agent(s), the combined workflow) and the final workflow with mapped actions. It follows the below format:
```
output_data = {
    "id": id,
    "user_query": user_query,
    "design_choices": design_choices,
    "images": images,
    "recruitment_status": recruitment_status,
    "workflows": workflows,
    "combined_workflow": combined_workflow,
    "final_workflow": retrieved_workflow
}
```


#### (2) Execution phase

To run the execution code, you need to be using a Darwin system and have <a href="https://www.adobe.com/creativecloud.html">Adobe Creative Cloud applications</a> downloaded in your Desktop.
```bash
python -u execution/executor_darwin.py \
  --input_path $PATH_TO_INPUT_FILE
```

Arguments for the executor code are as follows:
  - `--input_path`: Path to input data file.
  - Execution outcome will be saved according to the path referred in the planned workflow.

#### Executable Javascript files
Since Adobe Creative Cloud applications do not support direct API calls, we utilize their scripting environment to execute the actions. We provide manually written JavaScript codes that only require parameter values as inputs. Parameter keys for each action are defined based on Adobe’s scripting guides. Javascript for each Adobe application are in:

- **Photo Editor (Adobe Photoshop):** `execution/skillset/ps`
- **Vector Graphic Editor (Adobe Illustrator):** `execution/skillset/ai`
- **Layout Designer (Adobe InDesign):** `execution/skillset/id`



### Evaluation
#### (1) Workflow Evaluation

We define four evaluation criteria:
- **Delivery Rate:** This metric measures whether agents can successfully deliver a workflow within a limited number of steps. The step limit is determined by the difficulty level, based on the number of expert agents involved: (1) Easy: 1 expert, max 10 steps; (2) Medium: 2 experts, max 20 steps; (3) Hard: 3 experts, max 30 steps. Workflow that fall into dead loops or exceed the step limit are considered as failures.
- **Design Pass Rate:** This metric assesses whether agents can correctly incorporate both explicit design components specified in the user query and implicit commonsense constraints. We prompt GPT-5 to provide a score from 1 to 5 for each of the three aspects: color, text, and images.
- **Step Efficiency:** We measure the ratio of non-duplicate steps to the total number of steps.
- **Agent Use Efficiency:** Since a single workflow might involve multiple expert agents, we define efficiency as minimizing the frequency of switching between agents. A higher efficiency score indicates fewer transitions and better agent utilization.

To run evaluation of the planned workflow, run:
```bash
python -u evaluation/evaluation_workflow.py \
  --model $VLM
```
For `--model`, put the model name to evaluate.

  
#### (2) Execution Evaluation

We define five evaluation criteria:
- **Execution Success Rate:** This metric measures the success rate of execution attempts, calculated as the percentage of successful executions out of the total executions performed.
- **Fidelity:** A successful design should correctly incorporate the input images specified in the user query. We quantify fidelity using template matching from opencv library.
- **Content Similarity:** We measure the semantic similarity between the user query and the execution outcome using CLIPScore.
- **VQA Pass Rate:** We measure whether the execution outcome aligns with the design components in the user query using Visual Question Answering (VQA). We generate questions for each query by prompting GPT-4. We use a prompt LLaVA-1.5 7b to generate answers as Yes or No. The final pass rate is the average accuracy across all questions.
  - Generated questions for each design type is in: `data/with_questions`
- **Creativity:** We assess the creativity of our design outcomes along two axes: (1) Originality, which measures the uniqueness of the design, and (2) Elaboration, which measures the extent to which the design expands on the information in the user query by adding meaningful details. We prompt GPT-5 to provide a score from 1 to 5 for each axis.

To run evaluation of the execution outcomes, run:
```bash
python -u evaluation/evaluation_execution.py \
  --model $VLM
```
For `--model`, put the model name to evaluate.

---

<a id="citation"></a>
## 🤲 Citation
If you find our work useful in your research, please consider citing our work:
```
@misc{ki2025graphicbenchplanningbenchmarkgraphic,
      title={GraphicBench: A Planning Benchmark for Graphic Design with Language Agents}, 
      author={Dayeon Ki and Tianyi Zhou and Marine Carpuat and Gang Wu and Puneet Mathur and Viswanathan Swaminathan},
      year={2025},
      eprint={2504.11571},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2504.11571}, 
}
```

<a id="contact"></a>
## 📧 Contact
For questions, issues, or collaborations, please reach out to [dayeonki@umd.edu](mailto:dayeonki@umd.edu).

