# relation_extraction_BERT
fastapi relation extraction service in docker which uses BERT embeddings and  GPU acceleration

---

This repository contains a model for relation extraction in the Slovenian language. Repository for the method which was used for training the model
can be found on https://github.com/monologg/R-BERT. We used the [CroSloEngual](https://huggingface.co/EMBEDDIA/crosloengual-bert) BERT model to fine-tune the
model for our task.

## Project structure

- `src/` contains the script for predicting the relations and contains the source code of our work. fastapi service.
- `BERT_data.zip` contains our fine-tuned BERT model.


## Run with docker

To run this service we first need to extract the folder contained in BERT_data.zip into the root of this project.

#### Run GPU accelerated container 

 To run GPU accelerated docker containers you need to have an Nvidia GPU and [CUDA for WSL](https://docs.nvidia.com/cuda/wsl-user-guide/index.html) on Windows 10 or 11
 or [The NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) for Linux. 

 To build the docker image run:

 `docker build . -t bert_relation_extraction_gpu -f DockerfileGPU`

 To run the image in a GPU accelerated container use:
 
 ```
 docker run --rm -it --name bert_relation_extraction \
        --gpus=all \
        -e useGPU=True \
        --mount type=bind,source="$(pwd)"/BERT_data,target=/BERT_data,ro \
        -p:8000:8000 \
        bert_relation_extraction_gpu
  ```
 
#### Run normal container 


 To build the docker image run:

 `docker build . -t bert_relation_extraction -f Dockerfile`

 To run the image in a normal container use:

 ```
 docker run --rm -it --name bert_relation_extraction \
        --mount type=bind,source="$(pwd)"/BERT_data,target=/BERT_data,ro \
        -p:8000:8000 \
        bert_relation_extraction
  ```

 ## Run locally
 
 To run this project we recommend  python 3.8.
 
 First, we need to extract the folder contained in BERT_data.zip into the folder `src`.
 
 To install dependencies run `pip install -r requirements.txt -f https://download.pytorch.org/whl/cpu/torch_stable.html` in the root folder of this project.
 
 Run `uvicorn main:app --host 0.0.0.0 --port 8000` in the folder `src` to run the aplication on http://0.0.0.0:8000.

 #### Run with Nvidia CUDA

 For GPU acceleration you need to have the [CUDA toolkit](https://developer.nvidia.com/cuda-toolkit).

 To enable the GPU acceleration you will need to manually change the `use_gpu` parameter in `src/mark_entities.py` `classla.Pipeline` to `True`
 and string `device` in `src/predict.py` to `"cuda"`.
 
 ## Use
 
 Rest API is provided by FastAPI/uvicorn.
 
 After starting up the API, the OpenAPI/Swagger documentation will become accessible at http://localhost:8000/docs and http://localhost:8000/openapi.json.
 
 For extracting the relations in a sentence you can send get request to http://localhost:8000/find_relations/{text} where {text} represents the sentence.
 You can also send a get request to the http://localhost:8000/find_relations and add parameter `text` which contains the sentence. The return form for those
 two get requests can be found on http://localhost:8000/docs when the service is running.
 
 
 ## Use with your own BERT model

This service can be used with BERT models fine-tuned by method [R-BERT](https://github.com/monologg/R-BERT). To use this service with your model
you need to create your own `BERT_data` folder in the root of this project for docker use or in the folder `src` for local use. This folder
needs to have `pytorch_model.bin`, `training_args.bin` and `config.json` that you get from fine-tuning the BERT model with [R-BERT](https://github.com/monologg/R-BERT).
You also need to add the `vocab.txt` file from the BERT model and `properties-with-labels.txt` which has relation labels and descriptions. 
Examples for these files can be found in the `BERT_data.zip` file

**Note** This project uses NER tagger for the Slovenian language. If you want to use this project for another language you will need to change 
`src/change mark_entities_in_text.py` and perhaps dependencies in `requirements.txt`.


**Note** This project uses `transformers.AutoTokenizer.from_pretrained` function for BERT tokenization. If you use a BERT model with a different recommended tokenization
method you can change it in the `load_auto_tokenizer` function in `src/utils.py`.

**Note** Relations in `properties-with-labels.txt` should have the same order as in `labels.txt` in project [R-BERT](https://github.com/monologg/R-BERT)
 when you fine-tuned BERT model.
