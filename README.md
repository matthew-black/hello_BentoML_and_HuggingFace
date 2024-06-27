# BentoML to Play with Hugging Face Transformer Models

Based on:
* https://docs.bentoml.com/en/latest/get-started/quickstart.html

---

## What are the Components of This Open-Source Sorcery?

Here's all that's in `requirements.txt`:

```
bentoml
torch
transformers
```

To my understanding:
* `bentoml` (**BentoML**) is a framework for serving models as HTTP endpoints.
  * It, like FastAPI, has data-validation niceties built into it via **Pydantic**.
* `transformers`  is **Hugging Face**'s library that provides APIs for common NLP tasks (including text, image, and audio).
  * This specific app uses the `transformers.pipeline` API.
    * `pipeline` is a high-level abstraction that makes it easy to use a number of common models.
    * `pipeline("summarization")` instantiates a [SummarizationPipeline](https://huggingface.co/docs/transformers/v4.41.3/en/main_classes/pipelines#transformers.SummarizationPipeline).
* `torch` (**PyTorch**) is a framework for building deep learning models.
  * In this case, it's a dependency of `transformers`.
  * Without it installed, `transformers` spits out a warning, **and** an error when the summarization service fails to instantiate.
  ```
    RuntimeError: At least one of TensorFlow 2.0 or PyTorch should be installed.To install
    TensorFlow 2.0, read the instructions at https://www.tensorflow.org/install/ To install
    PyTorch, read the instructions at https://pytorch.org/.
    
    None of PyTorch, TensorFlow >= 2.0, or Flax have been found. Models won't be available
    and only tokenizers, configuration and file/data utilities can be used.
  ```

---

## Create and Instantiate a Virtual Environment:

<details>
  <summary>No need to Google this, future self. 🙂</summary>

  #### Create:

  * `python3 -m venv .venv`
    * `-m venv` = use the `venv` module
    * `.venv` = the name of the virtual environment folder

  ### Instantiate:

  * `source .venv/bin/activate`

  #### Verify:

  * `which python`
    * should output `/Users/..../.venv/bin/python`

  #### Deactivate:

  * `deactivate`

  #### Thank You:

  * https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/

</details>


---

## Install the Dependencies Specified by `requirements.txt`:

* `pip install -r requirements.txt`
  * `-r` just means "install stuff from a requirements file."

---

## Use BentoML to Serve the Model as an HTTP Server:

* `bentoml serve service:Summarization`
  * This instantiates a BentoML service on `localhost:3000`.

---

## Open `localhost:3000` in Your Browser 😱:

It's beautiful. OpenAPI for free!
![Auto OpenAPI Documentation](./images/openAPI.png)

---

## Hit the `/summarize` Endpoint w/ Postman:

![Example of Summarized Text](./images/summarized_chapter_one.png)

---

## On Trying Different Models:

* By default, this service uses `sshleifer/distilbart-cnn-12-6`.
* Could instead specify something, like so: 
  * `pipeline("summarization", model="facebook/bart-large-cnn")`
* Hugging Face makes it **very** easy to find and mess with different models:
  * https://huggingface.co/models?pipeline_tag=summarization&sort=downloads
* **Don't blow up your hard drive, future self!** 🙂
  * All downloaded models are stored here:
    * `.cache/huggingface/hub`
* Lastly, each model can have its own recommended usage
  * https://huggingface.co/pszemraj/led-base-book-summary

---

## Deployment Options:

* Automagically deploy a "Bento" to [BentoCloud](https://www.bentoml.com/cloud)
  * As long as you're [logged-in](https://docs.bentoml.com/en/latest/bentocloud/how-tos/manage-access-token.html), this is as simple as:
    * `bentoml deploy`
* Make your own "Bento" via [Docker containerization](https://docs.bentoml.com/en/latest/guides/containerization.html).
  * Then, deploy it anywhere that's compatible with Dockerized stuff.

---

## Thoughts Upon Exiting This Rabbit Hole:

It seems to me that making an awesome summarizing service would require a **lot** of pulling these kinds of levers:

  ```python
    result = self.summarization_pipeline(
        text,
        max_length=50,       # Try to keep it short
        min_length=30,       # Ensure it's not too short
        do_sample=True,      # Enable sampling
        temperature=0.7,     # Control randomness
        top_k=50,            # Consider top 50 tokens
        top_p=0.95,          # Nucleus sampling
        length_penalty=2.0,  # Penalize longer sequences
        num_beams=4,         # Use beam search
        early_stopping=True  # Stop early if possible
    )
  ```

But, not only that, **dynamically pulling these levers** based on information about the text to be summarized. (AKA: Length, source, genre, format, etc...)