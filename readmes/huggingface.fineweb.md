# 🍷 FineWeb
<center>
    <img src="https://huggingface.co/datasets/HuggingFaceFW/admin/resolve/main/fineweb-logo.png" alt="FineWeb: The finest collection of data the web has to offer">
</center>

> 15 trillion tokens of the finest data the 🌐 web has to offer

# Table of Contents
- [🍷 FineWeb](#-fineweb)
   * [What is it?](#what-is-it)
   * [What is being released?](#what-is-being-released)
   * [Changelog](#changelog)
   * [How to download and use 🍷 FineWeb](#how-to-download-and-use-🍷-fineweb)
      + [Using 🏭 `datatrove`](#using-datatrove)
      + [Using `huggingface_hub`](#using-huggingface_hub)
      + [Using `datasets`](#using-datasets)
   * [Breakdown by dump/crawl](#breakdown-by-dumpcrawl)
   * [Dataset performance evaluation and ablations](#dataset-performance-evaluation-and-ablations)
      + [Hyper-parameters for ablation models](#hyper-parameters-for-ablation-models)
      + [Ablation evaluation benchmarks](#ablation-evaluation-benchmarks)
      + [Comparison with other datasets](#comparison-with-other-datasets)
- [Dataset card for 🍷 FineWeb](#dataset-card-for-🍷-fineweb)
   * [Dataset Summary](#dataset-summary)
   * [Dataset Structure](#dataset-structure)
      + [Data Instances](#data-instances)
      + [Data Fields](#data-fields)
      + [Data Splits](#data-splits)
   * [Dataset Creation](#dataset-creation)
      + [Curation Rationale](#curation-rationale)
      + [Source Data](#source-data)
      + [Data processing steps](#data-processing-steps)
      + [Annotations](#annotations)
      + [Personal and Sensitive Information](#personal-and-sensitive-information)
   * [Considerations for Using the Data](#considerations-for-using-the-data)
      + [Social Impact of Dataset](#social-impact-of-dataset)
      + [Discussion of Biases](#discussion-of-biases)
      + [Other Known Limitations](#other-known-limitations)
   * [Additional Information](#additional-information)
      + [Licensing Information](#licensing-information)
      + [Future work](#future-work)
      + [Citation Information](#citation-information)

## What is it?

The 🍷 FineWeb dataset consists of more than **15T tokens** of cleaned and deduplicated english web data from CommonCrawl. The data processing pipeline is optimized for LLM performance and ran on the 🏭 [`datatrove`](https://github.com/huggingface/datatrove/) library, our large scale data processing library. 

🍷 FineWeb was originally meant to be a fully open replication of 🦅 [RefinedWeb](https://huggingface.co/papers/2306.01116), with a release of the **full dataset** under the **ODC-By 1.0 license**. However, by carefully adding additional filtering steps, we managed to push the performance of 🍷 FineWeb well above that of the original 🦅 RefinedWeb, and models trained on our dataset also outperform models trained on other commonly used high quality web datasets (like C4, Dolma-v1.6, The Pile, SlimPajama, RedPajam2) on our aggregate group of [benchmark tasks](https://huggingface.co/datasets/HuggingFaceFW/fineweb/blob/main/lighteval_tasks.py).

That said, we think there is still room for additional filtering and improvement and intend to continue exploring how to improve the dataset quality in coming versions of 🍷 FineWeb.

## What is being released?

Along with the dataset, which includes all CommonCrawl dumps since 2013, we also share all the code needed to fully reproduce our processing setup using the 🏭 [`datatrove`](https://github.com/huggingface/datatrove/) library [here](https://github.com/huggingface/datatrove/blob/main/examples/fineweb.py). To enable full replication of our results, we have also published the small ablation models we have trained using [`nanotron`](https://github.com/huggingface/nanotron/) to validate the dataset and compare it with other reference datasets. You will find them [here](https://huggingface.co/collections/HuggingFaceFW/ablation-models-662457b0d213e8c14fe47f32), with checkpoints every 1000 steps. We have also published our evaluation results [here](https://huggingface.co/datasets/HuggingFaceFW/fineweb/blob/main/eval_results.csv). Our evaluation setup is available [here](https://huggingface.co/datasets/HuggingFaceFW/fineweb/blob/main/lighteval_tasks.py).

You will find details on the different processing decisions we took and some interesting explorations of deduplication methods on our [blogpost](https://huggingface.co/spaces/HuggingFaceFW/blogpost-fineweb-v1).

## Changelog
_Previous versions remain available in the branch `version name`._

- **v1.1.0 (31-05-2024):** We reprocessed and reuploaded 11 dumps, `CC-MAIN-2021-49` to `CC-MAIN-2023-40`, as we found a bug on their deduplication. We also added the most recent dump: `CC-MAIN-2024-18`, crawled over April 2024. Expect a small perf improvement
- **v1.0.0 (21-04-2024):** Initial version

## How to download and use 🍷 FineWeb

You can load the full dataset or a specific crawl/dump (see table below). Dumps have the format `CC-MAIN-(year)-(week number)`.

### (Smaller) sample versions
Along with config `default` (all the data), and the configs for each individual dump, you can also download the following configs:
- `sample-350BT`: a subset randomly sampled from the whole dataset of around 350B gpt2 tokens (388GB)
- `sample-100BT`: a subset randomly sampled from the whole dataset of around 100B gpt2 tokens (277.4GB)
- `sample-10BT`: a subset randomly sampled from the whole dataset of around 10B gpt2 tokens (27.6GB)

`sample-10B` was sampled from `sample-100B` which in turn was sampled from `sample-350BT`.

### Using 🏭 [`datatrove`](https://github.com/huggingface/datatrove/)

```python
from datatrove.pipeline.readers import ParquetReader

# limit determines how many documents will be streamed (remove for all)
# to fetch a specific dump: hf://datasets/HuggingFaceFW/fineweb/data/CC-MAIN-2024-10
# replace "data" with "sample/100BT" to use the 100BT sample
data_reader = ParquetReader("hf://datasets/HuggingFaceFW/fineweb/data", limit=1000) 
for document in data_reader():
    # do something with document
    print(document)

###############################    
# OR for a processing pipeline:
###############################

from datatrove.executor import LocalPipelineExecutor
from datatrove.pipeline.readers import ParquetReader
from datatrove.pipeline.filters import LambdaFilter
from datatrove.pipeline.writers import JsonlWriter

pipeline_exec = LocalPipelineExecutor(
    pipeline=[
        # replace "data/CC-MAIN-2024-10" with "sample/100BT" to use the 100BT sample
        ParquetReader("hf://datasets/HuggingFaceFW/fineweb/data/CC-MAIN-2024-10", limit=1000),
        LambdaFilter(lambda doc: "hugging" in doc.text),
        JsonlWriter("some-output-path")
    ],
    tasks=10
)
pipeline_exec.run()
```

### Using `huggingface_hub`

```python
from huggingface_hub import snapshot_download
folder = snapshot_download(
                "HuggingFaceFW/fineweb", 
                repo_type="dataset",
                local_dir="./fineweb/",
                # replace "data/CC-MAIN-2023-50/*" with "sample/100BT/*" to use the 100BT sample
                allow_patterns="data/CC-MAIN-2023-50/*")
```

For faster downloads, make sure to install `pip install huggingface_hub[hf_transfer]` and set the environment variable `HF_HUB_ENABLE_HF_TRANSFER=1`.

### Using `datasets`

```python
from datasets import load_dataset
# use name="sample-10BT" to use the 10BT sample
fw = load_dataset("HuggingFaceFW/fineweb", name="CC-MAIN-2024-10", split="train", streaming=True)
```

## Breakdown by dump/crawl

| Dump | Time period | Disk size (GB) | gpt2 tokens (billions) |
| --- | --- | --- | --- |
| CC-MAIN-2024-18 | April 2024             | 417.6          | 154.4                       |
| CC-MAIN-2024-10 | February/March 2024    | 432.0          | 157.2                       |
| CC-MAIN-2023-50 | November/December 2023 | 650.0          | 239.7                       |
| CC-MAIN-2023-40 | September/October 2023 | 668.7          | 252.0                       |
| CC-MAIN-2023-23 | May/June 2023          | 654.4          | 249.2                       |
| CC-MAIN-2023-14 | March/April 2023       | 621.3          | 236.5                       |
| CC-MAIN-2023-06 | January/February 2023  | 621.9          | 233.9                       |
| CC-MAIN-2022-49 | November/December 2022 | 631.2          | 237.5                       |
| CC-MAIN-2022-40 | September/October 2022 | 606.4          | 228.7                       |
| CC-MAIN-2022-33 | August 2022            | 434.6          | 163.5                       |
| CC-MAIN-2022-27 | June/July 2022         | 574.9          | 216.1                       |
| CC-MAIN-2022-21 | May 2022               | 646.4          | 242.7                       |
| CC-MAIN-2022-05 | January 2022           | 520.1          | 195.4                       |
| CC-MAIN-2021-49 | November/December 2021 | 413.7          | 155.5                       |
| CC-MAIN-2021-43 | October 2021           | 601.5          | 221.0                       |
| CC-MAIN-2021-43 | October 2021 | 601.5 | 221.0 |
| CC-MAIN-2021-39 | September 2021 | 518.9 | 190.6 |
| CC-MAIN-2021-31 | July/August 2021 | 593.9 | 217.7 |
| CC-MAIN-2021-25 | June 2021 | 424.4 | 155.7 |
| CC-MAIN-2021-21 | May 2021 | 455.9 | 167.4 |
| CC-MAIN-2021-17 | April 2021 | 556.0 | 204.1 |
| CC-MAIN-2021-10 | February/March 2021 | 463.2 | 169.6 |
| CC-MAIN-2021-04 | January 2021 | 562.4 | 205.4 |
| CC-MAIN-2020-50 | November/December 2020 | 422.8 | 154.3 |
| CC-MAIN-2020-45 | October 2020 | 426.9 | 155.8 |
| CC-MAIN-2020-40 | September 2020 | 555.5 | 202.4 |
| CC-MAIN-2020-34 | August 2020 | 379.6 | 138.7 |
| CC-MAIN-2020-29 | July 2020 | 489.6 | 178.7 |
| CC-MAIN-2020-24 | May/June 2020 | 398.7 | 145.1 |
| CC-MAIN-2020-16 | March/April 2020 | 454.0 | 165.6 |
| CC-MAIN-2020-10 | February 2020 | 369.6 | 134.7 |
| CC-MAIN-2020-05 | January 2020 | 483.3 | 176.4 |
| CC-MAIN-2019-51 | December 2019 | 359.3 | 130.9 |
| CC-MAIN-2019-47 | November 2019 | 395.4 | 144.0 |
| CC-MAIN-2019-43 | October 2019 | 422.3 | 153.9 |
| CC-MAIN-2019-39 | September 2019 | 394.4 | 143.7 |
| CC-MAIN-2019-35 | August 2019 | 454.2 | 165.4 |
| CC-MAIN-2019-30 | July 2019 | 416.6 | 151.5 |
| CC-MAIN-2019-26 | June 2019 | 412.9 | 150.1 |
| CC-MAIN-2019-22 | May 2019 | 432.8 | 157.4 |
| CC-MAIN-2019-18 | April 2019 | 426.7 | 155.3 |
| CC-MAIN-2019-13 | March 2019 | 417.8 | 152.1 |
| CC-MAIN-2019-09 | February 2019 | 467.2 | 169.9 |
| CC-MAIN-2019-04 | January 2019 | 438.1 | 158.7 |
| CC-MAIN-2018-51 | December 2018 | 498.6 | 180.8 |
| CC-MAIN-2018-47 | November 2018 | 437.7 | 158.9 |
| CC-MAIN-2018-43 | October 2018 | 468.8 | 169.9 |
| CC-MAIN-2018-39 | September 2018 | 429.2 | 155.2 |
| CC-MAIN-2018-34 | August 2018 | 408.2 | 148.0 |
| CC-MAIN-2018-30 | July 2018 | 501.5 | 181.4 |
| CC-MAIN-2018-26 | June 2018 | 467.5 | 170.0 |
| CC-MAIN-2018-22 | May 2018 | 398.6 | 144.2 |
| CC-MAIN-2018-17 | April 2018 | 435.1 | 158.1 |
| CC-MAIN-2018-13 | March 2018 | 471.5 | 171.5 |
| CC-MAIN-2018-09 | February 2018 | 490.2 | 178.0 |
| CC-MAIN-2018-05 | January 2018 | 493.5 | 180.7 |
| CC-MAIN-2017-51 | December 2017 | 442.6 | 161.5 |
| CC-MAIN-2017-47 | November 2017 | 457.9 | 167.1 |
| CC-MAIN-2017-43 | October 2017 | 535.6 | 194.9 |
| CC-MAIN-2017-39 | September 2017 | 444.5 | 162.3 |
| CC-MAIN-2017-34 | August 2017 | 503.2 | 183.4 |
| CC-MAIN-2017-30 | July 2017 | 439.2 | 161.2 |
| CC-MAIN-2017-26 | June 2017 | 491.5 | 179.8 |
| CC-MAIN-2017-22 | May 2017 | 441.0 | 161.5 |
| CC-MAIN-2017-17 | April 2017 | 596.8 | 218.6 |
| CC-MAIN-2017-13 | March 2017 | 579.8 | 212.1 |
| CC-MAIN-2017-09 | February 2017 | 492.2 | 180.2 |
| CC-MAIN-2017-04 | January 2017 | 474.3 | 174.4 |
| CC-MAIN-2016-50 | December 2016 | 448.9 | 165.4 |
| CC-MAIN-2016-44 | October 2016 | 467.8 | 172.0 |
| CC-MAIN-2016-40 | September 2016 | 386.1 | 142.8 |
| CC-MAIN-2016-36 | August 2016 | 339.6 | 126.3 |
| CC-MAIN-2016-30 | July 2016 | 346.0 | 128.4 |
| CC-MAIN-2016-26 | June 2016 | 256.5 | 95.5 |
| CC-MAIN-2016-22 | May 2016 | 310.9 | 115.4 |
| CC-MAIN-2016-18 | April 2016 | 298.1 | 110.8 |
| CC-MAIN-2016-07 | February 2016 | 342.7 | 127.2 |
| CC-MAIN-2015-48 | November 2015 | 353.9 | 131.3 |
| CC-MAIN-2015-40 | September 2015 | 284.0 | 105.5 |
| CC-MAIN-2015-35 | August 2015 | 359.4 | 133.2 |
| CC-MAIN-2015-32 | July 2015 | 352.4 | 130.1 |
| CC-MAIN-2015-27 | June 2015 | 335.5 | 124.0 |
| CC-MAIN-2015-22 | May 2015 | 380.2 | 140.4 |
| CC-MAIN-2015-18 | April 2015 | 389.0 | 143.8 |
| CC-MAIN-2015-14 | March 2015 | 337.5 | 124.5 |
| CC-MAIN-2015-11 | February 2015 | 361.4 | 133.3 |
| CC-MAIN-2015-06 | January 2015 | 356.1 | 131.3 |
| CC-MAIN-2014-52 | December 2014 | 388.5 | 143.3 |
| CC-MAIN-2014-49 | November 2014 | 319.9 | 117.7 |
| CC-MAIN-2014-42 | October 2014 | 371.1 | 136.4 |
| CC-MAIN-2014-41 | September 2014 | 408.1 | 150.2 |
| CC-MAIN-2014-35 | August 2014 | 395.7 | 145.6 |
| CC-MAIN-2014-23 | July 2014 | 425.0 | 156.5 |
| CC-MAIN-2014-15 | April 2014 | 369.1 | 135.7 |
| CC-MAIN-2014-10 | March 2014 | 396.2 | 146.2 |
| CC-MAIN-2013-48 | Winter 2013 | 396.8 | 145.9 |
| CC-MAIN-2013-20 | Summer 2013 | 393.9 | 144.5 |
| Total |  | 43056.6 | 15835.2 |

## Dataset performance evaluation and ablations

We conducted our dataset performance ablations and evaluations by training a series of 1.8B parameters models on 27 billion tokens. To compare 🍷 FineWeb with other datasets, we also trained one of these 1.8B models per target dataset, on 350 billion tokens sampled from it (or the entire dataset when its size was < 350 billion tokens).

### Hyper-parameters for ablation models

The detailed configurations for training the 1.8B parameters ablation model can be found here (link will be added soon).

### Ablation evaluation benchmarks

To conduct the ablations for each of our dataset filtering choices, we selected a set of benchmarks which we identified as “high-signal” benchmarks. These benchmarks were selected according to the following criteria:

- small variance between runs trained on different samplings of the same dataset
- performance increasing monotically during training (or close)
- separation between runs on datasets of known quality (C4, The Pile, RedPajama) higher than the variance between runs with various modeling/data seeds

We used the following list of benchmark for our ablation runs:

- commonsense_qa (acc_norm)
- hellaswag (acc/acc_norm)
- openbookqa (acc/acc_norm)
- piqa (acc/acc_norm)
- siqa (acc/acc_norm)
- winogrande (acc/acc_norm)
- sciq (acc/acc_norm)
- arc (acc/acc_norm)
- mmlu (acc/acc_norm)

To compare runs we consider an aggregate score, the average of the scores for these tasks.

The prompts for all these benchmarks are formatted in order to compute and compare the log-likelihood of the full answers for each multiple choice question. All the implementation details for the benchmarks are available in `lighteval` [here](https://huggingface.co/datasets/HuggingFaceFW/fineweb/blob/main/lighteval_tasks.py).

### Comparison with other datasets

We compared 🍷 FineWeb with the following datasets:

- [RefinedWeb](https://huggingface.co/datasets/tiiuae/falcon-refinedweb)
- [C4](https://huggingface.co/datasets/allenai/c4)
- [Dolma v1.6](https://huggingface.co/datasets/allenai/dolma) (the CommonCrawl part)
- [The Pile](https://huggingface.co/datasets/EleutherAI/pile)
- [SlimPajama](https://huggingface.co/datasets/cerebras/SlimPajama-627B)
- [RedPajama2](https://huggingface.co/datasets/togethercomputer/RedPajama-Data-V2) (deduplicated)

You will find these models on [this collection](https://huggingface.co/collections/HuggingFaceFW/ablation-models-662457b0d213e8c14fe47f32). We have uploaded checkpoints at every 1000 training steps. You will also find our full [evaluation results here](https://huggingface.co/datasets/HuggingFaceFW/fineweb/blob/main/eval_results.csv).

<center>
    <img src="https://huggingface.co/datasets/HuggingFaceFW/admin/resolve/main/fineweb-ablations.png" alt="ablations">
</center>

_Note:_ The plot is smoothed by averaging 5k steps in a rolling window.

# Dataset card for 🍷 FineWeb

## Dataset Description

- **Homepage and Repository:** [https://huggingface.co/datasets/HuggingFaceFW/fineweb](https://huggingface.co/datasets/HuggingFaceFW/fineweb)
- **Point of Contact:** please create a discussion on the Community tab
- **License:** Open Data Commons Attribution License (ODC-By) v1.0

### Dataset Summary

This dataset was created by processing 96 [CommonCrawl](https://commoncrawl.org/) dumps comprising web data crawled from the summer of 2013 to April of 2024. 🍷 FineWeb includes a variety of domains and topics in English and is primarily intended to be used as a research artifact on public data in the context of pretraining dataset for large language models. The CommonCrawl data was carefully processed, filtered and deduplicated with the 🏭 [`datatrove`](https://github.com/huggingface/datatrove/) library, resulting in the largest publicly available clean LLM pretraining dataset, counting around 15 trillion tokens (gpt2 tokenizer).

## Dataset Structure

### Data Instances

The following is an example sample from the dataset. It is part of the `CC-MAIN-2021-43` and was crawled on `2021-10-15T21:20:12Z`.

```json
{
   "text": "This is basically a peanut flavoured cream thickened with egg yolks and then set into a ramekin on top of some jam. Tony, one of the Wedgwood chefs, suggested sprinkling on some toasted crushed peanuts at the end to create extra crunch, which I thought was a great idea. The result is excellent.",
   "id": "<urn:uuid:e5a3e79a-13d4-4147-a26e-167536fcac5d>",
   "dump": "CC-MAIN-2021-43",
   "url": "<http://allrecipes.co.uk/recipe/24758/peanut-butter-and-jam-creme-brulee.aspx?o_is=SimilarRecipes&o_ln=SimRecipes_Photo_7>",
   "date": "2021-10-15T21:20:12Z",
   "file_path": "s3://commoncrawl/crawl-data/CC-MAIN-2021-43/segments/1634323583083.92/warc/CC-MAIN-20211015192439-20211015222439-00600.warc.gz",
   "language": "en",
   "language_score": 0.948729,
   "token_count": 69
}
```

### Data Fields

- `text` (string): the main text content
- `id` (string): original unique identifier for this sample from CommonCrawl
- `dump` (string): the CommonCrawl dump this sample was a part of
- `url` (string): url to the original page where `text` was present
- `date` (string): crawl date (from CommonCrawl)
- `file_path` (string): s3 path for the individual CommonCrawl warc file containing this sample
- `language` (string): `en` for all the samples in this dataset
- `language_score` (float): language prediction score (`0.01.0`) as reported by the [fastText language classifier](https://github.com/huggingface/datatrove/blob/main/src/datatrove/pipeline/filters/language_filter.py)
- `token_count` (int): number of tokens when applying the `gpt2` tokenizer to this sample

### Data Splits

The `default` subset includes the entire dataset. If you would like to only use the data from a particular [CommonCrawl dump](https://commoncrawl.org/overview), you can use the dump name as a subset. You will find the full list of available dumps on the table above.
From experiments we have run, not all dumps give the same performance. For relatively small trainings (<550 billion tokens) we recommend using the recent `CC-MAIN-2023-50`, `CC-MAIN-2024-10` and `CC-MAIN-2024-18`. 

## Dataset Creation

### Curation Rationale

While multiple open-weights models have regularly been released in recent months, these releases often do not include the model's training data. With 🍷 FineWeb we aim to provide the open source community with a very large clean pretraining dataset that can be used to push the envelope on truly open source models (open source models where data is also released). 

### Source Data

The source data consists of webpages crawled by the CommonCrawl foundation over the 2013-2024 time period.

We then extracted the main page text from the html of each webpage, carefully filtered each sample and deduplicated each individual CommonCrawl dump/crawl.

While we originally intended to deduplicate the dataset as a whole, our ablations showed that training on a sampling of individually deduplicated dumps/crawls outperformed training on a sampling of all the dumps/crawls deduplicated together. You will find more details on our [blogpost](https://huggingface.co/spaces/HuggingFaceFW/blogpost-fineweb-v1).

### Data processing steps

We used the 🏭 `datatrove` library to process the data.
You can find a **working script** that launches the [entire processing pipeline here](https://github.com/huggingface/datatrove/blob/main/examples/fineweb.py).

The data processing pipeline consists of:

1. [Url Filtering](https://github.com/huggingface/datatrove/blob/9a88bebc86a554f8521faa70b12ad4fa0c227537/src/datatrove/pipeline/filters/url_filter.py), removing documents originating from Malicious and NSFW websites, using both block-list as well as subwords detection
2. [Trafilatura](https://github.com/huggingface/datatrove/blob/9a88bebc86a554f8521faa70b12ad4fa0c227537/src/datatrove/pipeline/extractors/trafilatura.py) text extraction on the raw HTML from CommonCrawl’s warc files
3. [FastText LanguageFilter](https://github.com/huggingface/datatrove/blob/9a88bebc86a554f8521faa70b12ad4fa0c227537/src/datatrove/pipeline/filters/language_filter.py), removing any document with `en` language score lower than **0.65**
4. Quality filtering
    1. [Gopher Repetition /](https://github.com/huggingface/datatrove/blob/9a88bebc86a554f8521faa70b12ad4fa0c227537/src/datatrove/pipeline/filters/gopher_repetition_filter.py) [Quality](https://github.com/huggingface/datatrove/blob/9a88bebc86a554f8521faa70b12ad4fa0c227537/src/datatrove/pipeline/filters/gopher_quality_filter.py)
    2. [C4 Quality filters](https://github.com/huggingface/datatrove/blob/9a88bebc86a554f8521faa70b12ad4fa0c227537/src/datatrove/pipeline/filters/c4_quality_filter.py) except `terminal_punct` rule
    3. [FineWeb custom filters](https://github.com/huggingface/datatrove/blob/05194d3960741e7d5c0bd0d6dd69d44514622549/src/datatrove/pipeline/filters/fineweb_quality_filter.py), consisting of heuristics for removing list-like documents, documents with repeated lines and documents with likely wrong line formatting. 
5. [MinHash deduplication](https://github.com/huggingface/datatrove/blob/6daa5e879e06b21e6886b37e2b1be4ae58a658b6/src/datatrove/pipeline/dedup/minhash.py) with each crawl deduplicated individually (5-grams, 14x8 hash functions)
6. [PII Formatting](https://github.com/huggingface/datatrove/blob/main/src/datatrove/pipeline/formatters/pii.py) to anonymize email and public IP addresses

### Annotations

We augment the original samples with the `language`, `language_score` and `token_count` annotations. The language related annotations are automatically generated by our [language filter](https://github.com/huggingface/datatrove/blob/main/src/datatrove/pipeline/filters/language_filter.py). `token_count` is generated by [applying the gpt2 tokenizer](https://github.com/huggingface/datatrove/blob/main/src/datatrove/pipeline/tokens/counter.py) to the `text` column.

### Personal and Sensitive Information

We anonymize email addresses and public IP addresses. 

For emails, we apply a regex pattern and replace any occurrence of an email address with either `email@example.com` or `firstname.lastname@example.org`. For IP addresses, we also employ a regex pattern and then further filter to only anonymize IP addresses [allocated for public networks](https://www.iana.org/assignments/iana-ipv4-special-registry/iana-ipv4-special-registry.xhtml). Matched IP addresses are then replaced with one of the following randomly generated IP addresses, which at the time of dataset creation were not responding to ping requests: `22.214.171.124`, `126.96.36.199`, `188.8.131.52`, `184.108.40.206`, `220.127.116.11`, and `18.104.22.168`. We decided against applying regex patterns for phone numbers due to the high false positive rate.

Despite our efforts, given that 🍷 FineWeb is sourced from the internet at large, it is very likely that some personable identifiable information (PII) will be present. If you find your own PII in 🍷 FineWeb and would like it removed, please fill out our [PII removal form](https://forms.gle/VyNT3ZAUPZjPuWp39).

## Considerations for Using the Data

### Social Impact of Dataset

With the release of this dataset we aim to make model training more accessible to the machine learning community at large. 

While multiple open-weights models with strong performance have been publicly released in the past, more often than not these releases are not accompanied by the corresponding training dataset. This is unfortunate as the dataset specificities and characteristics have been demonstrated to have a very large impact and role in the performances of the models. As the creation of a high quality training dataset is a fundamental requirement to training an LLM capable of excelling at downstream tasks, with 🍷 FineWeb we (a) not only make the dataset creation process more transparent, by sharing our entire processing setup including the codebase used, we also (b) help alleviate the costs of dataset curation, both in time and in compute, for model creators by publicly releasing our dataset with the community.

### Discussion of Biases

Efforts were made to minimize the amount of NSFW and toxic content present in the dataset by employing filtering on the URL level. However, there are still a significant number of documents present in the final dataset that could be considered toxic or contain harmful content. As 🍷 FineWeb was sourced from the web as a whole, any harmful biases typically present in it may be reproduced on our dataset.

We deliberately avoided using machine learning filtering methods that define text quality based on the similarity to a “gold” source such as wikipedia or toxicity classifiers as these methods have been known to [disproportionately remove content in specific dialects](https://aclanthology.org/D16-1120/) and [overclassify as toxic text related to specific social identities](https://arxiv.org/pdf/2109.07445.pdf), respectively.

### Other Known Limitations

As a consequence of some of the filtering steps applied, it is likely that code content is not prevalent in our dataset. If you are training a model that should also perform code tasks, we recommend you use 🍷 FineWeb with a code dataset, such as [The Stack v2](https://huggingface.co/datasets/bigcode/the-stack-v2). You should also probably consider complementing 🍷 FineWeb with specialized curated sources (such as Wikipedia, for example) as they will likely have better formatting than the wikipedia content included in 🍷 FineWeb (we did not tailor the processing to individual websites).

## Additional Information

### Licensing Information

The dataset is released under the **Open Data Commons Attribution License (ODC-By) v1.0** [license](https://opendatacommons.org/licenses/by/1-0/). The use of this dataset is also subject to [CommonCrawl's Terms of Use](https://commoncrawl.org/terms-of-use).

### Future work

We plan to not only continue but also expand our efforts to create open-source high quality training datasets and to improve 🍷 FineWeb itself in future iterations.

### Citation Information

```
@software{penedo2024fineweb,
  author = {Penedo, Guilherme and Kydlíček, Hynek and von Werra, Leandro and Wolf, Thomas},
  title = {FineWeb},
  month = April,
  year = 2024,
  doi = { 10.57967/hf/2493 },
  url = {https://huggingface.co/datasets/HuggingFaceFW/fineweb}
}

```
