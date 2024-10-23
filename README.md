# DOC Story Generation V2

## Installation

Tested using Python 3.9 but similar versions should also work.

```
pip install -r requirements.txt
pip install -e .
```

By default we use VLLM to serve models. 
You'll need to make a one-line change to the VLLM package to get their API server to work with logprobs requests that are used for reranking.
In your install of VLLM (you can find it using e.g., `pip show vllm`), find the line at https://github.com/vllm-project/vllm/blob/acbed3ef40f015fcf64460e629813922fab90380/vllm/entrypoints/openai/api_server.py#L177 (your exact line number might vary slightly depending on VLLM version) and change the `p` at the end to e.g., `max(p, -1e8)`. This will avoid an error related to passing jsons back from the server, due to json not handling inf values.

## Getting Started

We divide the story generation procedure into 3 steps, Premise, Plan, and Story. Everything will be run from the `scripts` directory:

```
cd scripts
```

Everything will read the information from the `defaults` configuration in `config.yaml` unless specified otherwise using the `--configs` flag.

See the corresponding `config.yaml` for details on options for each step of the pipeline. You'll have to fill in the particular model you want to use (marked TODO in each `config.yaml`). This system was mainly tested with LLaMA2-7B-Chat and ChatGPT, with the default options given; several other options are supported but not as heavily tested. When changing the model, make sure you also change `server_type` and `prompt_format` as needed. You can also add new options directly to the config as needed; you can also see the main prompts in `prompts.json`.

By default we use VLLM to serve models. Run the generation pipeline.

```
python {premise/plan/story}/generate.py
```

By default, files are written to the `output/` folder. Premise and Plan are formatted as jsons which can be edited for human interaction.

## Some Known Issues / Potential Improvements

- When start multiple model servers for different models, we should allocate them to different GPUs or load on multi-GPU as needed. 
- During plan generation, the model likes to overgenerate characters when determining which characters appear in a given plot point.
- Diversity of premise / plan ideas is kind of bad when using chat models, since they like to generate the same ideas over and over. Can try to increase temperature, or other ways to increase diversity, ideally without sacrificing quality.
- We should implement vaguest-first expansion (using a model to predict which node is the most vague) rather than breadth-first expansion when creating the outline during plan generation.
- We should test `text-davinci-003` for generating passages during the story generation stage since it's a completion model; the OpenAI Chat API is very limiting.
- Some model types and some more obscure options in the code aren't well-tested. Please let us know if you run into any issues.

## Commands:

路径：/root/autodl-tmp/doc-storygen-v2/scripts

1. python premise/generate.py

- 输入：对话框，1-2句英文
- 输出：output/premise.json，有title和premise两个字段，允许编辑

2. python plan/generate.py

- 输入：output/premise.json
- 输出：output/plan.json，需要树形json展示，允许编辑

3. python story/generate.py

- 输入：output/plan.json
- 输出：output/story.txt
