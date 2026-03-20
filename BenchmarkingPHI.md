# Benchmarking Private Home Intelligence

This document explains how I benchmarked models running via Ollama on a Desktop with an RTX3090.

**Overview:** I have a home setup comprising of AI models running on a Linux Desktop with a RTX3090 via Ollama. It serves as the intelligence hubs. Individual users run [Jan.AI](https://jan.ai) as the agentic client application to interact with the model. This setup is similar to having ChatGPT/Gemini/Claude running within the home. Of course the models are smaller and therefore likely to be less capable. However, my actual experience is that the setup produces better quality results than the lower tier Gemini Plus plan and the free ChatGPT plan.


## Benchmarking Overview

We use the [Inspect](https://inspect.aisi.org.uk/) open-source framework to run all benchmarks. This framework was 
created by the UK AI Security institute to ease the evaluation of Large Language Models.

Using this framework we run the following benchmarks:

| Benchmark | Description | Best for Testing... |
| --- | --- | --- |
| GAIA | Tests "Google-proof" tasks: researching the web, handling files, and multi-step reasoning. | General Assistant Tasks |
| WebArena | A sandbox where the agent must navigate realistic websites (e-commerce, forums) to find info. | Web Research |
| SWE-Bench | A sandbox where the agent must navigate realistic websites (e-commerce, forums) to find info. | Software/Code Agents |
| τ²-bench (Tau2) | Simulates a business environment where the agent must use APIs and talk to a user to solve a problem. | Business/Tool Use |

## Benchmarking setup

Create a directory for bench marking (eg. JanAIBench). From within that directory create a Python virtual environment. 

Create a Huggingface token.

```
python3 -m venv .venv
source .venv/bin/activate
pip install inspect-ai inspect-evals openai
export HF_TOKEN=YOUR_HF_TOKEN
```

The above should be sufficient. However, if you encounter errors then you can use the text below (paste it into a requirements.txt) and do 

```
aioboto3==15.5.0
aiobotocore==2.25.1
aiofiles==25.1.0
aiohappyeyeballs==2.6.1
aiohttp==3.13.3
aioitertools==0.13.0
aiosignal==1.4.0
annotated-doc==0.0.4
annotated-types==0.7.0
anyio==4.12.1
attrs==26.1.0
backoff==2.2.1
beautifulsoup4==4.14.3
boto3==1.40.61
botocore==1.40.61
certifi==2026.2.25
charset-normalizer==3.4.6
click==8.2.1
datasets==4.8.2
debugpy==1.8.20
dill==0.4.1
distro==1.9.0
docstring_parser==0.17.0
filelock==3.25.2
frozenlist==1.8.0
fsspec==2025.9.0
h11==0.16.0
hf-xet==1.4.2
httpcore==1.0.9
httpx==0.28.1
huggingface_hub==1.7.1
idna==3.11
ijson==3.5.0
inspect_ai==0.3.199
inspect_evals==0.5.1
Jinja2==3.1.6
jiter==0.13.0
jmespath==1.1.0
jsonlines==4.0.0
jsonpatch==1.33
jsonpath-ng==1.8.0
jsonpointer==3.0.0
jsonref==1.1.0
jsonschema==4.26.0
jsonschema-specifications==2025.9.1
linkify-it-py==2.1.0
markdown-it-py==4.0.0
MarkupSafe==3.0.3
mdit-py-plugins==0.5.0
mdurl==0.1.2
mmh3==5.2.1
multidict==6.7.1
multiprocess==0.70.19
nest-asyncio2==1.7.2
numpy==2.4.3
openai==2.29.0
packaging==26.0
pandas==3.0.1
pathlib_abc==0.5.2
pillow==12.1.1
platformdirs==4.9.4
propcache==0.4.1
psutil==7.2.2
pyarrow==23.0.1
pydantic==2.12.5
pydantic_core==2.41.5
Pygments==2.19.2
python-dateutil==2.9.0.post0
python-dotenv==1.2.2
PyYAML==6.0.3
referencing==0.37.0
regex==2026.2.28
requests==2.32.5
rich==14.3.3
rpds-py==0.30.0
s3fs==2025.9.0
s3transfer==0.14.0
semver==3.0.4
shellingham==1.5.4
shortuuid==1.0.13
six==1.17.0
sniffio==1.3.1
soupsieve==2.8.3
tenacity==9.1.4
textual==8.1.1
tiktoken==0.12.0
toml==0.10.2
tqdm==4.67.3
typer==0.24.1
typing-inspection==0.4.2
typing_extensions==4.15.0
uc-micro-py==2.0.0
universal_pathlib==0.3.10
urllib3==2.6.3
wrapt==1.17.3
xxhash==3.6.0
yarl==1.23.0
zipp==3.23.0
zstandard==0.25.0
```

**Example commandline to run inspect:**
```bash
inspect eval inspect_evals/gaia_level1 \
  --model openai/qwen3.5:9b \
  --limit 1 \
  --max-connections 1 \
  --timeout 600 \
  --model-base-url http://192.168.3.152:11434/v1 \
  --env OPENAI_API_KEY=ollama
  ```

Running inspect with an agent does not work:
```bash
inspect eval inspect_evals/gaia_level1 \
  --model openai/qwen3.5:9b \
  --model-base-url http://192.168.3.152:11434/v1 \
  --env OPENAI_API_KEY=ollama \
  --solver agent \
  --tools python,search \
  --limit 1
```


## Hardware Specification

| Component | Specification |
| --- | --- |
| Motherboard | Asus TUF GAMING B550M-PLUS WIFI II Micro ATX AM4 Motherboard |
| GPU | Nvidia RTX3090 (INNO3D GeForce RTX 3090) | 
| CPU | AMD Ryzen 5 5600 3.5GHz 6 Core |
| Memory | 32GB: Corsair Vengeance LPX 16gb DDR4-3600 CL18 Memory - 2nos |
| Storage | Samsung 980 1TB PCI- E 3.0 NVME M.2-2280 INTERNAL Solid State Drive (MZ-V8V1T0BW) |
| SMPS | Cooler Master 850 watts gold smps |
| Cabinet | Cooler Master TD500 |
| Fan | Cabinet fan Noctua NF-F12-PWM 120MM 4pin 1500 rpm |